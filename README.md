# Firebase Notification Demo – Android Studio

Dự án này là một ví dụ cơ bản minh họa cách tích hợp **Firebase Cloud Messaging (FCM)** vào ứng dụng Android, bao gồm cả việc nhận và xử lý **Notification Messages** và **Data Messages** trong các trạng thái foreground và background.

---

## 📌 Mục lục
1. [Firebase là gì?](#-firebase-là-gì)  
2. [Cơ chế hoạt động của Firebase Notification](#-cơ-chế-hoạt-động-của-firebase-notification)  
   - [Notification Messages](#1-notification-messages)  
   - [Data Messages](#2-data-messages)  
   - [Khác biệt giữa Foreground và Background](#-foreground-vs-background)  
3. [Cấu hình dự án Android Studio](#-cấu-hình-dự-án-android-studio)  
   - [1. Thêm Firebase vào dự án](#1-thêm-firebase-vào-dự-án)  
   - [2. Cập nhật `build.gradle`](#2-cập-nhật-buildgradle)  
   - [3. Cấu hình `AndroidManifest.xml`](#3-cấu-hình-androidmanifestxml)  
4. [Triển khai trong code](#-triển-khai-trong-code)  
   - [MainActivity – Lấy FCM Token](#1-mainactivity--lấy-fcm-token)  
   - [MyFirebaseMessagingService – Xử lý tin nhắn](#2-myfirebasemessagingservice--xử-lý-tin-nhắn)  
5. [Test với Postman](#-test-với-postman)  
6. [Tóm tắt luồng hoạt động](#-tóm-tắt-luồng-hoạt-động)  
7. [Tài liệu tham khảo](#-tài-liệu-tham-khảo)  

---

## 🔥 Firebase là gì?
**Firebase** là một nền tảng phát triển ứng dụng di động và web của Google. Trong đó, **Firebase Cloud Messaging (FCM)** là dịch vụ giúp gửi tin nhắn thông báo từ server đến ứng dụng Android/iOS/web **miễn phí**.

---

## 📩 Cơ chế hoạt động của Firebase Notification

### 1. Notification Messages
- Chứa trường `notification` trong payload.  
- Android **tự động hiển thị thông báo** khi app ở background.  
- Ví dụ payload:
  ```json
  {
    "token": "<DEVICE_FCM_TOKEN>",
    "notification": {
      "title": "Test Title",
      "body": "Test Body"
    }
  }
  ```
- Nếu app ở foreground → cần xử lý thủ công trong `onMessageReceived()`.

---

### 2. Data Messages
- Chỉ chứa trường `data`.  
- Luôn **được xử lý trong code** (kể cả foreground hay background).  
- Ví dụ payload:
  ```json
  {
     "token": "<DEVICE_FCM_TOKEN>",
     "data": {
        "name": "Phatdepzai",
        "title": "Test Title",
        "body": "Test Body"
     }
  }
  ```
- Ưu điểm: Linh hoạt, có thể tự định nghĩa dữ liệu.

---

### ⚖️ Foreground vs Background
| Loại message          | Foreground (ứng dụng mở) | Background (ứng dụng bị ẩn) |
|-----------------------|--------------------------|-----------------------------|
| **Notification**      | Gọi `onMessageReceived()`, cần tự build thông báo | Android tự hiển thị thông báo |
| **Data**              | Gọi `onMessageReceived()`, tự xử lý | Gọi `onMessageReceived()`, tự xử lý |

👉 Vì vậy ta cần **tự viết Service** (`MyFirebaseMessagingService`) để xử lý cả hai loại, đảm bảo thống nhất trải nghiệm người dùng.

---

## ⚙️ Cấu hình dự án Android Studio

### 1. Thêm Firebase vào dự án
- Tạo project trên [Firebase Console](https://console.firebase.google.com).  
- Thêm app Android bằng **ApplicationId** (`com.example.notificationfirebase`).  
- Tải file `google-services.json` và đặt vào thư mục `app/`.

---

### 2. Cập nhật `build.gradle`

**File cấp project (`build.gradle.kts`):**
```kotlin
plugins {
    alias(libs.plugins.android.application) apply false
    alias(libs.plugins.kotlin.android) apply false
    alias(libs.plugins.kotlin.compose) apply false
    alias(libs.plugins.google.gms.google.services) apply false
}
```

**File cấp module app (`app/build.gradle.kts`):**
```kotlin
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.kotlin.compose)
    alias(libs.plugins.google.gms.google.services)
}

dependencies {
    implementation(libs.firebase.messaging)
}
```

---

### 3. Cấu hình `AndroidManifest.xml`
```xml
<service
    android:name=".MyFirebaseMessagingService"
    android:exported="false">
    <intent-filter>
        <action android:name="com.google.firebase.MESSAGING_EVENT"/>
    </intent-filter>
</service>

<meta-data
    android:name="com.google.firebase.messaging.default_notification_channel_id"
    android:value="@string/default_notification_channel_id" />
```

- `service`: Đăng ký service để xử lý tin nhắn.  
- `meta-data`: Tạo **channel id** (bắt buộc với Android 8+).

---

## 💻 Triển khai trong code

### 1. MainActivity – Lấy FCM Token
```kotlin
FirebaseMessaging.getInstance().token.addOnCompleteListener { task ->
    if (!task.isSuccessful) {
        Log.w("MyApp", "Fetching FCM registration token failed", task.exception)
        return@addOnCompleteListener
    }
    val token = task.result
    Log.d("MyApp", "FCM Token: $token")
}
```
- Token này dùng để gửi notification đến thiết bị qua **Postman** hoặc server.

---

### 2. MyFirebaseMessagingService – Xử lý tin nhắn
```kotlin
override fun onMessageReceived(remoteMessage: RemoteMessage) {
    val notification = remoteMessage.notification
    val data = remoteMessage.data

    if (notification != null) {
        // Notification Message
        val title = notification.title
        val body = notification.body ?: ""
        sendNotification(title, body)
    } else if (data.isNotEmpty()) {
        // Data Message
        val title = data["title"]
        val name = data["name"] ?: "User"
        val body = data["body"] ?: ""
        val customizedBody = "Hello $name\n$body"
        sendNotification(title, customizedBody)
    }
}
```

---

## 🧪 Test với Postman
1. Copy **FCM token** từ logcat.  
2. Gửi POST request đến:  
   ```
   https://fcm.googleapis.com/v1/projectId/messages:send
   ```
3. Headers:
   ```
   Authorization: Bearer access_token
   Content-Type: application/json
   ```
4. Body ví dụ:
   ```json
   {
     "message": {
       "token": REGISTRATION_TOKEN (FCM TOKEN),
       "notification": {
         "title": "FCM API test",
         "body": "This is the body of the notification.",
         "image": "https://cat.10515.net/1.jpg"
       }
     }
   }
   ```

---

## 🔄 Tóm tắt luồng hoạt động
1. App khởi động → **lấy FCM Token**.  
2. Server/Postman gửi tin nhắn đến FCM với token đó.  
3. Firebase push message đến app.  
4. - Nếu là **Notification Message** + background → Android tự hiển thị.  
   - Nếu là **Foreground Notification Message** hoặc **Data Message** → gọi `onMessageReceived()` → app tự build thông báo.  

