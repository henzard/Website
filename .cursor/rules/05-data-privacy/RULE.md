---
description: "GDPR and POPIA compliance, data privacy, and user rights implementation"
alwaysApply: true
---

# Data Privacy & Compliance (GDPR/POPIA)

## Core Privacy Principles

### 1. Lawfulness, Fairness, and Transparency
- Users must know what data is collected and why
- Clear, accessible privacy policy required
- No hidden data collection

### 2. Purpose Limitation
- Collect data only for specified purposes
- Don't repurpose data without consent

### 3. Data Minimization
- Collect only necessary data
- Don't store data "just in case"

### 4. Accuracy
- Keep data up-to-date
- Allow users to correct their data

### 5. Storage Limitation
- Don't keep data longer than necessary
- Implement data retention policies

### 6. Integrity and Confidentiality
- Secure data properly
- Prevent unauthorized access

### 7. Accountability
- Document compliance measures
- Maintain audit trails

## User Rights Implementation

### Right to Access (Data Export)

```typescript
// services/privacyService.ts

interface UserDataExport {
  profile: User;
  bookings: Booking[];
  preferences: UserPreferences;
  activityLog: ActivityLog[];
  exportedAt: string;
}

export class PrivacyService {
  /**
   * Export all user data in machine-readable format (GDPR Article 20)
   */
  async exportUserData(userId: string): Promise<UserDataExport> {
    // Verify user is authenticated and authorized
    const currentUser = auth.currentUser;
    if (!currentUser || currentUser.uid !== userId) {
      throw new Error('Unauthorized');
    }

    // Collect all user data from various collections
    const [profile, bookings, preferences, activityLog] = await Promise.all([
      userRepository.findById(userId),
      bookingRepository.findByUser(userId),
      preferencesRepository.findByUser(userId),
      activityRepository.findByUser(userId),
    ]);

    const exportData: UserDataExport = {
      profile: this.sanitizeForExport(profile),
      bookings: bookings.map(b => this.sanitizeForExport(b)),
      preferences: this.sanitizeForExport(preferences),
      activityLog: activityLog.map(a => this.sanitizeForExport(a)),
      exportedAt: new Date().toISOString(),
    };

    // Log the export for audit trail
    await this.logDataAccess(userId, 'DATA_EXPORT');

    return exportData;
  }

  /**
   * Generate downloadable file for user
   */
  async generateExportFile(userId: string): Promise<Blob> {
    const data = await this.exportUserData(userId);
    
    // Provide in multiple formats
    const jsonBlob = new Blob(
      [JSON.stringify(data, null, 2)], 
      { type: 'application/json' }
    );

    return jsonBlob;
  }

  private sanitizeForExport(data: any) {
    // Remove internal system fields
    const { __typename, _metadata, ...cleanData } = data;
    return cleanData;
  }
}
```

### Right to Erasure (Data Deletion)

```typescript
/**
 * Delete user account and all associated data (GDPR Article 17)
 */
export class AccountDeletionService {
  async deleteUserAccount(userId: string, reason?: string): Promise<void> {
    // Verify authentication
    const currentUser = auth.currentUser;
    if (!currentUser || currentUser.uid !== userId) {
      throw new Error('Unauthorized');
    }

    // Start deletion process
    await this.initiateAccountDeletion(userId, reason);
  }

  private async initiateAccountDeletion(userId: string, reason?: string) {
    const batch = writeBatch(db);

    // 1. Mark account as pending deletion (30-day grace period)
    const userRef = doc(db, 'users', userId);
    batch.update(userRef, {
      status: 'pending_deletion',
      deletionRequestedAt: Timestamp.now(),
      deletionReason: reason || 'User requested',
    });

    // 2. Log the deletion request
    await this.logDataDeletion(userId, 'DELETION_REQUESTED');

    // 3. Schedule actual deletion after grace period
    await this.scheduleAccountDeletion(userId);

    await batch.commit();

    // 4. Send confirmation email
    await this.sendDeletionConfirmationEmail(userId);
  }

  /**
   * Permanent deletion (called by Cloud Function after grace period)
   */
  async permanentlyDeleteAccount(userId: string): Promise<void> {
    // Delete from all collections
    await Promise.all([
      this.deleteUserProfile(userId),
      this.deleteUserBookings(userId),
      this.deleteUserPreferences(userId),
      this.deleteUserFiles(userId),
      this.deleteUserActivity(userId),
      this.anonymizeUserReviews(userId), // Don't delete reviews, anonymize
    ]);

    // Delete authentication account
    await admin.auth().deleteUser(userId);

    // Final audit log (keep for legal compliance)
    await this.logDataDeletion(userId, 'PERMANENTLY_DELETED');
  }

  private async deleteUserProfile(userId: string) {
    const userRef = doc(db, 'users', userId);
    await deleteDoc(userRef);

    // Delete subcollections
    const subcollections = ['private', 'notifications', 'sessions'];
    for (const subcollection of subcollections) {
      await this.deleteCollection(`users/${userId}/${subcollection}`);
    }
  }

  private async deleteUserFiles(userId: string) {
    const storage = getStorage();
    const userFilesRef = ref(storage, `users/${userId}`);
    
    // List and delete all user files
    const fileList = await listAll(userFilesRef);
    await Promise.all(
      fileList.items.map(item => deleteObject(item))
    );
  }

  private async anonymizeUserReviews(userId: string) {
    // Don't delete reviews, just anonymize them
    const reviewsRef = collection(db, 'reviews');
    const q = query(reviewsRef, where('userId', '==', userId));
    const snapshot = await getDocs(q);

    const batch = writeBatch(db);
    snapshot.docs.forEach(doc => {
      batch.update(doc.ref, {
        userId: 'DELETED_USER',
        userName: 'Deleted User',
        userAvatar: null,
      });
    });

    await batch.commit();
  }

  private async deleteCollection(path: string) {
    const collectionRef = collection(db, path);
    const snapshot = await getDocs(collectionRef);
    
    const batch = writeBatch(db);
    snapshot.docs.forEach(doc => batch.delete(doc.ref));
    await batch.commit();
  }
}
```

### UI Components for User Rights

```typescript
// components/PrivacyControls.tsx

export const PrivacyControls: React.FC = () => {
  const { user } = useAuth();
  const [loading, setLoading] = useState(false);

  const handleExportData = async () => {
    setLoading(true);
    try {
      const blob = await privacyService.generateExportFile(user.id);
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = `my-data-${new Date().toISOString()}.json`;
      a.click();
      URL.revokeObjectURL(url);
      
      toast.success('Your data has been exported');
    } catch (error) {
      toast.error('Failed to export data');
    } finally {
      setLoading(false);
    }
  };

  const handleDeleteAccount = async () => {
    const confirmed = await confirmDialog({
      title: 'Delete Account',
      message: 'Are you sure? This action cannot be undone. Your account will be deleted after 30 days.',
      confirmText: 'Delete My Account',
      cancelText: 'Cancel',
      variant: 'danger',
    });

    if (!confirmed) return;

    try {
      await accountDeletionService.deleteUserAccount(user.id);
      toast.success('Account deletion scheduled. You have 30 days to cancel.');
      logout();
    } catch (error) {
      toast.error('Failed to delete account');
    }
  };

  return (
    <div className="space-y-6">
      <section>
        <h2 className="text-xl font-semibold mb-4">Your Privacy Rights</h2>
        
        <div className="space-y-4">
          {/* Export Data */}
          <Card>
            <h3 className="font-medium mb-2">Download Your Data</h3>
            <p className="text-sm text-gray-600 mb-4">
              Get a copy of all your data in a machine-readable format
            </p>
            <Button 
              onClick={handleExportData} 
              loading={loading}
              leftIcon={<DownloadIcon />}
            >
              Export My Data
            </Button>
          </Card>

          {/* Delete Account */}
          <Card>
            <h3 className="font-medium mb-2">Delete Your Account</h3>
            <p className="text-sm text-gray-600 mb-4">
              Permanently delete your account and all associated data
            </p>
            <Button 
              onClick={handleDeleteAccount}
              variant="danger"
              leftIcon={<TrashIcon />}
            >
              Delete My Account
            </Button>
          </Card>
        </div>
      </section>
    </div>
  );
};
```

## Consent Management

### Cookie Consent Banner

```typescript
// components/CookieConsent.tsx

interface ConsentPreferences {
  necessary: boolean; // Always true
  analytics: boolean;
  marketing: boolean;
  preferences: boolean;
}

export const CookieConsentBanner: React.FC = () => {
  const [showBanner, setShowBanner] = useState(false);
  const [showDetails, setShowDetails] = useState(false);

  useEffect(() => {
    const consent = localStorage.getItem('cookie-consent');
    if (!consent) {
      setShowBanner(true);
    }
  }, []);

  const handleAcceptAll = () => {
    const consent: ConsentPreferences = {
      necessary: true,
      analytics: true,
      marketing: true,
      preferences: true,
    };
    saveConsent(consent);
    setShowBanner(false);
  };

  const handleRejectAll = () => {
    const consent: ConsentPreferences = {
      necessary: true,
      analytics: false,
      marketing: false,
      preferences: false,
    };
    saveConsent(consent);
    setShowBanner(false);
  };

  const saveConsent = (consent: ConsentPreferences) => {
    localStorage.setItem('cookie-consent', JSON.stringify(consent));
    localStorage.setItem('cookie-consent-date', new Date().toISOString());
    
    // Initialize analytics based on consent
    if (consent.analytics) {
      initializeAnalytics();
    }
  };

  if (!showBanner) return null;

  return (
    <div className="fixed bottom-0 left-0 right-0 bg-white border-t shadow-lg p-4 z-50">
      <div className="max-w-6xl mx-auto">
        <p className="text-sm mb-4">
          We use cookies to enhance your experience. By continuing to visit this site you agree to our use of cookies.
          <a href="/privacy-policy" className="text-primary-600 ml-1">Learn more</a>
        </p>
        
        <div className="flex gap-2">
          <Button onClick={handleAcceptAll} variant="primary">
            Accept All
          </Button>
          <Button onClick={handleRejectAll} variant="outline">
            Reject All
          </Button>
          <Button onClick={() => setShowDetails(true)} variant="ghost">
            Customize
          </Button>
        </div>
      </div>
    </div>
  );
};
```

## Data Retention Policy

```typescript
// Cloud Function to enforce retention policy
export const enforceDataRetention = functions.pubsub
  .schedule('every 24 hours')
  .onRun(async (context) => {
    const now = Timestamp.now();
    const retentionPeriods = {
      activityLogs: 90,      // 90 days
      sessions: 30,          // 30 days
      deletedAccounts: 30,   // 30 days grace period
      anonymousData: 365,    // 1 year
    };

    // Delete old activity logs
    const oldLogs = await db
      .collection('activityLogs')
      .where('createdAt', '<', daysAgo(retentionPeriods.activityLogs))
      .get();

    const batch = db.batch();
    oldLogs.docs.forEach(doc => batch.delete(doc.ref));
    await batch.commit();

    // Process account deletions after grace period
    const accountsToDelete = await db
      .collection('users')
      .where('status', '==', 'pending_deletion')
      .where('deletionRequestedAt', '<', daysAgo(retentionPeriods.deletedAccounts))
      .get();

    for (const doc of accountsToDelete.docs) {
      await accountDeletionService.permanentlyDeleteAccount(doc.id);
    }
  });
```

## Audit Logging

```typescript
// Every data access must be logged
export const auditLogger = {
  async logAccess(userId: string, action: string, resource: string) {
    await addDoc(collection(db, 'auditLogs'), {
      userId,
      action,
      resource,
      timestamp: Timestamp.now(),
      ipAddress: await this.getClientIP(),
    });
  },
};
```

## Required Legal Pages

1. **Privacy Policy** - `/privacy-policy`
2. **Terms of Service** - `/terms`
3. **Cookie Policy** - `/cookies`
4. **Data Processing Agreement** - `/dpa` (for B2B)

## AI Decision Gate

**Before handling user data:**
1. ✅ Is data collection necessary and documented?
2. ✅ Is user consent obtained?
3. ✅ Can users export their data?
4. ✅ Can users delete their data?
5. ✅ Is data encrypted at rest and in transit?
6. ✅ Is audit logging implemented?
7. ✅ Is data retention policy enforced?
8. ✅ Are privacy controls easily accessible?

**Never collect data without consent. Never prevent users from accessing or deleting their data.**

