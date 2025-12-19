---
description: "Firebase Storage patterns for file uploads, image handling, and security"
alwaysApply: false
globs: ["**/storage/**", "**/upload/**", "**/components/*Upload*"]
---

# File Upload & Storage Standards

## 🎯 Core Principles

- ✅ Use Firebase Storage for all file uploads
- ✅ Validate file type and size on client AND server
- ✅ Generate unique filenames (prevent overwrites)
- ✅ Optimize images before upload
- ✅ Show upload progress
- ✅ Clean up orphaned files

---

## 🔒 Firebase Storage Security Rules

```javascript
// storage.rules
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    // User profile images
    match /users/{userId}/profile/{filename} {
      allow read: if true; // Public profile images
      allow write: if request.auth.uid == userId
                   && request.resource.size < 5 * 1024 * 1024 // 5MB
                   && request.resource.contentType.matches('image/.*');
    }
    
    // Private user documents
    match /users/{userId}/documents/{filename} {
      allow read, write: if request.auth.uid == userId
                        && request.resource.size < 10 * 1024 * 1024; // 10MB
    }
    
    // Public assets
    match /public/{allPaths=**} {
      allow read: if true;
      allow write: if request.auth != null
                   && request.resource.size < 5 * 1024 * 1024;
    }
  }
}
```

---

## 📤 Upload Service

```typescript
// services/storageService.ts
import { 
  ref, 
  uploadBytesResumable, 
  getDownloadURL,
  deleteObject,
  UploadTask 
} from 'firebase/storage';
import { storage } from '@/lib/firebase';
import { v4 as uuidv4 } from 'uuid';

interface UploadOptions {
  path: string;
  file: File;
  onProgress?: (progress: number) => void;
  metadata?: Record<string, string>;
}

interface UploadResult {
  url: string;
  path: string;
  filename: string;
}

export const uploadFile = async ({
  path,
  file,
  onProgress,
  metadata,
}: UploadOptions): Promise<UploadResult> => {
  // Generate unique filename
  const ext = file.name.split('.').pop();
  const filename = `${uuidv4()}.${ext}`;
  const fullPath = `${path}/${filename}`;
  
  const storageRef = ref(storage, fullPath);
  
  const uploadTask = uploadBytesResumable(storageRef, file, {
    contentType: file.type,
    customMetadata: metadata,
  });

  return new Promise((resolve, reject) => {
    uploadTask.on(
      'state_changed',
      (snapshot) => {
        const progress = (snapshot.bytesTransferred / snapshot.totalBytes) * 100;
        onProgress?.(progress);
      },
      (error) => reject(error),
      async () => {
        const url = await getDownloadURL(uploadTask.snapshot.ref);
        resolve({ url, path: fullPath, filename });
      }
    );
  });
};

export const deleteFile = async (path: string): Promise<void> => {
  const storageRef = ref(storage, path);
  await deleteObject(storageRef);
};
```

---

## 🖼️ Image Upload Component

```typescript
// components/ImageUpload.tsx
import { useState, useCallback } from 'react';
import { useDropzone } from 'react-dropzone';
import { uploadFile } from '@/services/storageService';

interface ImageUploadProps {
  path: string;
  onUpload: (url: string, path: string) => void;
  maxSize?: number;
  accept?: string[];
}

export const ImageUpload: React.FC<ImageUploadProps> = ({
  path,
  onUpload,
  maxSize = 5 * 1024 * 1024,
  accept = ['image/jpeg', 'image/png', 'image/webp'],
}) => {
  const [preview, setPreview] = useState<string | null>(null);
  const [progress, setProgress] = useState(0);
  const [isUploading, setIsUploading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const onDrop = useCallback(async (acceptedFiles: File[]) => {
    const file = acceptedFiles[0];
    if (!file) return;

    // Validate
    if (!accept.includes(file.type)) {
      setError('Invalid file type');
      return;
    }
    if (file.size > maxSize) {
      setError(`File must be less than ${Math.round(maxSize / 1024 / 1024)}MB`);
      return;
    }

    setError(null);
    setPreview(URL.createObjectURL(file));
    setIsUploading(true);

    try {
      const result = await uploadFile({
        path,
        file,
        onProgress: setProgress,
      });
      onUpload(result.url, result.path);
    } catch (err) {
      setError('Upload failed. Please try again.');
      setPreview(null);
    } finally {
      setIsUploading(false);
      setProgress(0);
    }
  }, [path, onUpload, maxSize, accept]);

  const { getRootProps, getInputProps, isDragActive } = useDropzone({
    onDrop,
    accept: { 'image/*': accept.map(t => `.${t.split('/')[1]}`) },
    maxFiles: 1,
  });

  return (
    <div className="space-y-2">
      <div
        {...getRootProps()}
        className={`border-2 border-dashed rounded-lg p-6 text-center cursor-pointer ${
          isDragActive ? 'border-primary-500 bg-primary-50' : 'border-gray-300'
        }`}
      >
        <input {...getInputProps()} />
        
        {preview ? (
          <img src={preview} alt="Preview" className="max-h-40 mx-auto rounded" />
        ) : (
          <div>
            <p>Drag image here or click to select</p>
            <p className="text-sm text-gray-500">Max {Math.round(maxSize / 1024 / 1024)}MB</p>
          </div>
        )}
      </div>

      {isUploading && (
        <div className="w-full bg-gray-200 rounded-full h-2">
          <div
            className="bg-primary-600 h-2 rounded-full transition-all"
            style={{ width: `${progress}%` }}
          />
        </div>
      )}

      {error && <p className="text-red-600 text-sm">{error}</p>}
    </div>
  );
};
```

---

## 🗜️ Image Optimization

```typescript
// utils/imageOptimization.ts

export const compressImage = async (
  file: File,
  maxWidth: number = 1200,
  quality: number = 0.8
): Promise<Blob> => {
  return new Promise((resolve, reject) => {
    const img = new Image();
    img.onload = () => {
      const canvas = document.createElement('canvas');
      let { width, height } = img;
      
      if (width > maxWidth) {
        height = (height * maxWidth) / width;
        width = maxWidth;
      }
      
      canvas.width = width;
      canvas.height = height;
      
      const ctx = canvas.getContext('2d');
      ctx?.drawImage(img, 0, 0, width, height);
      
      canvas.toBlob(
        (blob) => blob ? resolve(blob) : reject(new Error('Compression failed')),
        'image/jpeg',
        quality
      );
    };
    img.onerror = reject;
    img.src = URL.createObjectURL(file);
  });
};

// Usage before upload
const optimizedBlob = await compressImage(file, 800, 0.7);
const optimizedFile = new File([optimizedBlob], file.name, { type: 'image/jpeg' });
```

---

## ✅ Checklist

```
Security
☐ Storage rules configured
☐ File type validation (client + rules)
☐ File size limits enforced
☐ User can only access own files

Upload Experience
☐ Progress indicator shown
☐ Error messages displayed
☐ Preview before upload
☐ Cancel upload option
☐ Drag and drop support

Performance
☐ Images compressed before upload
☐ Unique filenames generated
☐ Orphaned files cleaned up

IF ANY ☐ UNCHECKED → Fix before deploying
```

---

**Remember: "Validate everything. Trust nothing. Show progress."** 📁✨

