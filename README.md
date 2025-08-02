# Hung Duy Distribution - App Updates 📱

> **Hệ thống quản lý phiên bản và thông báo cập nhật cho ứng dụng Hung Duy Distribution**

## 📋 Tổng quan

Repository này chứa **Appcast feed** để quản lý việc thông báo cập nhật ứng dụng cho cả iOS (TestFlight) và Android (Play Store). Hệ thống cho phép:

- ✅ **Kiểm tra phiên bản tự động** khi app khởi động
- ✅ **Thông báo optional updates** (người dùng có thể bỏ qua)  
- ✅ **Force updates** cho phiên bản quan trọng (bắt buộc cập nhật)
- ✅ **Hướng dẫn cập nhật** qua TestFlight hoặc Play Store
- ✅ **Release notes** tiếng Việt cho mỗi phiên bản

## 🏗️ Kiến trúc hệ thống

```
📱 Hung Duy App
├── 🔍 Version Check tại SplashScreen
├── 📡 Fetch appcast.xml từ GitHub
├── 🆚 So sánh version với pubspec.yaml
└── 🎯 Hành động:
    ├── ✅ No update → Continue normal flow
    ├── 📢 Optional update → Show dialog, allow continue
    └── 🚫 Force update → Block app, redirect to store
```

## 📁 Cấu trúc Files

```
hung-duy-app-updates/
├── README.md                    # 📖 Documentation này
├── appcast.xml                  # 📡 Main appcast feed
└── versions/                    # 📚 Lịch sử versions (optional)
    ├── v1.0.0.xml
    ├── v1.0.1.xml
    └── ...
```

## 🚀 Setup Ban đầu (One-time)

### 1. **App Configuration**

Trong app Flutter, đã được cấu hình:

- **Service**: `UpgraderService` quản lý logic upgrade
- **Wrapper**: `UpgraderWrapper` hiển thị dialogs  
- **Integration**: Tích hợp vào `SplashScreen` để check version đầu tiên

### 2. **GitHub Repository Setup**

Repository này (`Cat1m/hung-duy-app-updates`) đã được tạo với:
- ✅ **Public access** (required để app có thể fetch)
- ✅ **Raw access** cho appcast.xml
- ✅ **Auto-deployment** qua GitHub Pages (optional)

### 3. **URLs được sử dụng**

```bash
# Raw appcast URL (app sẽ fetch từ đây)
https://raw.githubusercontent.com/Cat1m/hung-duy-app-updates/refs/heads/main/appcast.xml

# TestFlight link
https://testflight.apple.com/join/FRjPt2Dk

# Play Store link  
https://play.google.com/store/apps/details?id=vn.com.hungduy.hung_duy_distribution
```

## 🔄 Quy trình Release Phiên bản mới

### 📋 **Checklist trước khi release:**

- [ ] Code đã được test kỹ
- [ ] Version được tăng trong `pubspec.yaml`
- [ ] Release notes đã được chuẩn bị
- [ ] TestFlight/Play Store builds đã sẵn sàng

### 🔢 **Step 1: Tăng version trong App**

**File: `pubspec.yaml`**
```yaml
# VÍ DỤ: Từ version hiện tại
version: 1.0.1+27

# Tăng lên version mới
version: 1.0.2+28
#        ^^^^^ ^^
#        |     └── Build number (luôn tăng)
#        └── Version name (tăng khi có features mới)
```

**Quy tắc versioning:**
- **Build number** (`+XX`): Luôn tăng lên (+1) cho mỗi build
- **Version name** (`X.Y.Z`): 
  - `Z` (patch): Bug fixes, minor improvements
  - `Y` (minor): New features, significant updates  
  - `X` (major): Breaking changes, major rewrites

### 📱 **Step 2: Build và upload lên stores**

**iOS (TestFlight):**
```bash
flutter build ios --release
# Upload to App Store Connect → TestFlight
```

**Android (Play Store):**
```bash
flutter build appbundle --release
# Upload to Google Play Console → Internal Testing/Production
```

### 📝 **Step 3: Update appcast.xml**

1. **Vào GitHub repository**: https://github.com/Cat1m/hung-duy-app-updates

2. **Click vào file `appcast.xml`**

3. **Click nút "Edit" (✏️)**

4. **Cập nhật version mới** (thay thế item đầu tiên):

```xml
<!-- THAY ĐỔI PHẦN NÀY -->
<item>
    <title>Phiên bản 1.0.2+28</title>  <!-- ← UPDATE VERSION -->
    <description>
        🎉 Cập nhật mới:
        • Sửa lỗi đăng nhập trên Android 14
        • Cải thiện hiệu suất tải dữ liệu  
        • Thêm tính năng xuất báo cáo Excel
        • Hỗ trợ dark mode cho toàn bộ app
        • Sửa lỗi crash khi quét QR code
    </description>
    <pubDate>Mon, 04 Aug 2025 10:00:00 +0000</pubDate>  <!-- ← UPDATE DATE -->
    
    <!-- iOS - TestFlight -->
    <enclosure url="https://testflight.apple.com/join/FRjPt2Dk" 
              sparkle:version="1.0.2"                    <!-- ← UPDATE -->
              sparkle:shortVersionString="1.0.2+28"      <!-- ← UPDATE -->
              sparkle:os="ios" />
              
    <!-- Android - Play Store -->
    <enclosure url="https://play.google.com/store/apps/details?id=vn.com.hungduy.hung_duy_distribution" 
              sparkle:version="1.0.2"                    <!-- ← UPDATE -->
              sparkle:shortVersionString="1.0.2+28"      <!-- ← UPDATE -->
              sparkle:os="android" />
</item>
```

5. **Commit changes** với message: `Release v1.0.2+28`

### ⚡ **Step 4: Verification**

Sau khi commit, kiểm tra:

1. **Raw URL works**: https://raw.githubusercontent.com/Cat1m/hung-duy-app-updates/refs/heads/main/appcast.xml

2. **App detection**: Mở app với version cũ → Sẽ thấy dialog upgrade

3. **Logs check**: Xem Flutter logs để confirm
```
upgrader: appStoreVersion: 1.0.2
upgrader: installedVersion: 1.0.1  
upgrader: shouldDisplayUpgrade: true ✅
```

## 🚨 Force Updates (Critical Updates)

Khi có **security fixes** hoặc **breaking changes**, cần force update:

### **Option 1: Sử dụng minimumSystemVersion**

```xml
<item>
    <title>Phiên bản 1.1.0+30 (Bắt buộc)</title>
    <description>
        🚨 Cập nhật bảo mật quan trọng:
        • Sửa lỗi bảo mật nghiêm trọng
        • Cập nhật API endpoint mới (bắt buộc)
        • ...
    </description>
    
    <!-- THÊM dòng này để force update -->
    <sparkle:minimumSystemVersion>1.1.0</sparkle:minimumSystemVersion>
    
    <enclosure url="..." sparkle:version="1.1.0" ... />
</item>
```

### **Option 2: Sử dụng critical update flag**

```xml
<enclosure url="..." 
          sparkle:version="1.1.0"
          sparkle:shortVersionString="1.1.0+30"
          sparkle:criticalUpdate="true"     <!-- ← Force update -->
          sparkle:os="ios" />
```

**Kết quả**: App sẽ hiển thị dialog bắt buộc, không cho phép dismiss, chỉ có nút "Cập nhật ngay".

## 🛠️ Troubleshooting

### ❓ **App không detect version mới**

**Check list:**
1. ✅ `appcast.xml` đã commit và push lên GitHub?
2. ✅ Raw URL accessible? Test trong browser
3. ✅ Version trong XML đúng format `X.Y.Z`?
4. ✅ App có internet connection?
5. ✅ Clear app cache: Xóa app → Reinstall

**Debug logs:**
```
# Enable debug trong UpgraderService
debugLogging: true

# Expected logs:
upgrader: download: https://raw.githubusercontent.com/...
upgrader: response statusCode: 200  
upgrader: appStoreVersion: 1.0.2
upgrader: installedVersion: 1.0.1
upgrader: shouldDisplayUpgrade: true
```

### ❓ **Dialog không hiển thị**

**Possible causes:**
1. **Navigator context issue**: Check UpgraderWrapper placement
2. **Version same**: App đã cùng version với appcast  
3. **User ignored**: User đã tap "Ignore" cho version này
4. **Too soon**: Chưa đủ thời gian từ lần alert trước (default: 3 days)

**Solutions:**
```dart
// Force show for testing
debugDisplayAlways: true,

// Reset ignored versions
await Upgrader.clearSavedSettings();
```

### ❓ **XML format errors**

**Common mistakes:**
```xml
<!-- ❌ WRONG -->
<sparkle:version>1.0.2+28</sparkle:version>

<!-- ✅ CORRECT -->  
<sparkle:version>1.0.2</sparkle:version>
<sparkle:shortVersionString>1.0.2+28</sparkle:shortVersionString>
```

**Validation**: Use online XML validators như xmllint hoặc các tools online.

## 📚 Version History Template

Để track lịch sử, có thể tạo folder `versions/`:

```
versions/
├── v1.0.0.md    # Release notes cho v1.0.0
├── v1.0.1.md    # Release notes cho v1.0.1
└── v1.0.2.md    # Release notes cho v1.0.2
```

**Example: `versions/v1.0.2.md`**
```markdown
# Version 1.0.2+28
**Release Date**: August 4, 2025
**Type**: Minor Update

## 🎉 New Features
- Thêm tính năng xuất báo cáo Excel
- Hỗ trợ dark mode cho toàn bộ app

## 🐛 Bug Fixes  
- Sửa lỗi đăng nhập trên Android 14
- Sửa lỗi crash khi quét QR code

## ⚡ Improvements
- Cải thiện hiệu suất tải dữ liệu
- Tối ưu giao diện người dùng

## 📱 Deployment
- **TestFlight**: Build uploaded on Aug 3, 2025
- **Play Store**: Released to Production on Aug 4, 2025
- **Rollout**: 100% users
```

## 🔧 Advanced Configuration

### **Custom Update Messages**

Trong app code, có thể custom messages:

```dart
// lib/core/services/upgrader/upgrader_service.dart
class _HungDuyUpgraderMessages extends UpgraderMessages {
  @override
  String get title => 'Có phiên bản mới! 🚀';
  
  @override  
  String get body => 'Phiên bản {{newVersion}} với nhiều tính năng mới!';
  
  // Custom theo business logic
}
```

### **Conditional Updates**

```xml
<!-- Chỉ update cho iOS -->
<enclosure sparkle:os="ios" ... />

<!-- Chỉ update cho Android -->  
<enclosure sparkle:os="android" ... />

<!-- Update cho tất cả platforms -->
<enclosure sparkle:os="ios" ... />
<enclosure sparkle:os="android" ... />
```

## 👥 Team Workflow

### **Roles & Responsibilities**

**🧑‍💻 Developers:**
- Update version trong `pubspec.yaml`
- Build và upload lên stores
- Test upgrade flow locally

**📱 QA Team:**
- Test upgrade scenarios
- Verify dialogs display correctly
- Check store links work

**🚀 Release Manager:**
- Update `appcast.xml` trên GitHub
- Monitor rollout metrics
- Handle rollback if needed

### **Release Schedule**

**📅 Typical Timeline:**
- **Monday**: Code freeze, version bump
- **Tuesday**: Build & upload to stores
- **Wednesday**: Store review & approval
- **Thursday**: Update appcast.xml → Live release
- **Friday**: Monitor & hotfix if needed

## 📞 Support

**🆘 Emergency Rollback:**
1. Revert `appcast.xml` về version trước
2. Commit immediately  
3. Monitor logs cho version detection

**📧 Contact:**
- **Developer**: [Team Developer]
- **Release Manager**: [Team Lead]
- **Repository**: https://github.com/Cat1m/hung-duy-app-updates

---

## 📝 Quick Reference

### **Release Checklist Template**

```markdown
## Release v1.0.X+XX Checklist

### Pre-Release
- [ ] Code reviewed & approved
- [ ] Version bumped in pubspec.yaml  
- [ ] Release notes prepared
- [ ] iOS build uploaded to TestFlight
- [ ] Android build uploaded to Play Store
- [ ] Builds approved in stores

### Release  
- [ ] Update appcast.xml with new version
- [ ] Verify raw URL accessible
- [ ] Test upgrade dialog on old version
- [ ] Monitor logs for errors

### Post-Release
- [ ] Verify users receiving updates
- [ ] Monitor crash rates
- [ ] Track adoption metrics
- [ ] Document any issues
```

### **Quick Commands**

```bash
# Check raw appcast  
curl https://raw.githubusercontent.com/Cat1m/hung-duy-app-updates/refs/heads/main/appcast.xml

# Validate XML
xmllint --noout appcast.xml

# Version bump (example)
sed -i 's/version: 1\.0\.1+27/version: 1.0.2+28/' pubspec.yaml
```

---

**🎉 Happy Releasing! Chúc team release thành công!** 🚀
