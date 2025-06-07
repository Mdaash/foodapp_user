# دليل ربط البحث بالباك إند

## نظرة عامة
تم تطوير نظام بحث متطور يدعم البحث المحلي والبحث من الباك إند مع تحسينات الأداء.

## الملفات المحدثة

### 1. SearchService (`lib/services/search_service.dart`)
خدمة شاملة للبحث تتضمن:
- البحث مع debouncing لتحسين الأداء
- نظام cache للنتائج
- دعم البحث باللغة العربية والإنجليزية
- إحصائيات البحث
- اقتراحات البحث التلقائي

### 2. SearchScreen المحدثة
- واجهة مستخدم عربية بالكامل مع `Directionality.rtl`
- بحث فوري محلي + بحث متقدم من الباك إند
- مؤشرات تحميل ذكية
- دعم البحث الجزئي والتطبيع للنص العربي

## كيفية ربط الباك إند

### 1. تحديث URL الباك إند
في ملف `lib/services/search_service.dart`:
```dart
static const String baseUrl = 'https://your-api-domain.com/api';
```
استبدل `your-api-domain.com` بالدومين الخاص بك.

### 2. API Endpoints المطلوبة

#### أ. البحث في المتاجر
```
GET /api/search/stores
```
**المعاملات:**
- `q`: نص البحث (مطلوب)
- `lang`: اللغة (افتراضي: ar)
- `category`: فئة المتجر (اختياري)
- `lat`: خط العرض (اختياري)
- `lng`: خط الطول (اختياري)
- `radius`: نطاق البحث بالكيلومتر (اختياري)

**الاستجابة:**
```json
{
  "stores": [
    {
      "id": "store_id",
      "name": "اسم المتجر",
      "category": "فئة المتجر",
      "image": "url_للصورة",
      "logoUrl": "url_للشعار",
      "rating": "4.5",
      "reviews": "100+",
      "distance": "2.5 km",
      "time": "25 min",
      "fee": "0$",
      "promo": "عرض خاص",
      "isOpen": true,
      "address": "العنوان"
    }
  ]
}
```

#### ب. البحث الشائع
```
GET /api/search/popular
```
**الاستجابة:**
```json
{
  "popular_searches": [
    "برجر",
    "بيتزا",
    "دجاج مقلي"
  ]
}
```

#### ج. اقتراحات البحث
```
GET /api/search/suggestions?q={query}
```
**الاستجابة:**
```json
{
  "suggestions": [
    "برجر كينج",
    "برجر هاوس",
    "برجر لاونج"
  ]
}
```

#### د. إحصائيات البحث
```
POST /api/search/analytics
```
**البيانات المرسلة:**
```json
{
  "query": "نص البحث",
  "results_count": 5,
  "timestamp": "2023-12-01T10:30:00Z"
}
```

### 3. إعداد Headers

```dart
final headers = {
  'Content-Type': 'application/json',
  'Accept': 'application/json',
  'Accept-Language': 'ar',
  // يمكن إضافة authentication headers
  // 'Authorization': 'Bearer $token',
};
```

### 4. إدارة الأخطاء

```dart
try {
  final response = await http.get(uri, headers: headers);
  
  if (response.statusCode == 200) {
    final data = json.decode(response.body);
    return processData(data);
  } else if (response.statusCode == 429) {
    // Rate limiting
    throw Exception('تم تجاوز الحد المسموح للطلبات');
  } else {
    throw Exception('خطأ في الخادم: ${response.statusCode}');
  }
} on SocketException {
  throw Exception('لا يوجد اتصال بالإنترنت');
} on TimeoutException {
  throw Exception('انتهت مهلة الاتصال');
} catch (e) {
  throw Exception('خطأ غير متوقع: $e');
}
```

## الميزات المطبقة

### ✅ تم الإنجاز
- [x] اتجاه عربي صحيح في شاشة البحث
- [x] البحث المحلي المحسّن مع تطبيع النص العربي
- [x] البحث الجزئي للكلمات
- [x] Debouncing لتحسين الأداء
- [x] نظام Cache للنتائج
- [x] مؤشرات التحميل الذكية
- [x] دمج النتائج المحلية مع الباك إند
- [x] إحصائيات البحث
- [x] البحث الشائع من الباك إند

### 🔄 قيد التطوير
- [ ] Authentication و Authorization
- [ ] البحث الجغرافي (Location-based)
- [ ] فلترة متقدمة
- [ ] التخزين المؤقت الذكي

## كيفية الاختبار

### 1. اختبار البحث المحلي
- يعمل فوراً مع المتاجر الموجودة في التطبيق
- يدعم البحث العربي والإنجليزي
- ترتيب النتائج حسب الصلة

### 2. اختبار ربط الباك إند
```bash
# اختبار API endpoints
curl -X GET "https://your-domain.com/api/search/stores?q=برجر" \
  -H "Accept-Language: ar"

curl -X GET "https://your-domain.com/api/search/popular" \
  -H "Accept-Language: ar"
```

### 3. مراقبة الأداء
- تحقق من Console logs للأخطاء
- مراقبة زمن الاستجابة
- تتبع معدل نجاح الطلبات

## المتطلبات

### Dependencies
```yaml
dependencies:
  http: ^1.1.0  # للطلبات HTTP
  
# تأكد من وجود هذه في pubspec.yaml
```

### Permissions (Android)
```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

### Permissions (iOS)
```xml
<!-- ios/Runner/Info.plist -->
<key>NSAppTransportSecurity</key>
<dict>
  <key>NSAllowsArbitraryLoads</key>
  <true/>
</dict>
```

## نصائح للتطوير

### 1. تحسين الأداء
- استخدم Debouncing مع 500ms
- فعّل Cache للنتائج الشائعة
- اضبط timeout المناسب للطلبات

### 2. تجربة المستخدم
- اعرض النتائج المحلية فوراً
- أضف مؤشرات تحميل واضحة
- وفر رسائل خطأ مفيدة

### 3. الأمان
- استخدم HTTPS دائماً
- فعّل authentication إذا لزم الأمر
- اعمل rate limiting من جانب الخادم

## الدعم

للمساعدة أو الاستفسارات، يرجى مراجعة:
- [وثائق Flutter HTTP](https://docs.flutter.dev/cookbook/networking/fetch-data)
- [دليل البحث الذكي](https://flutter.dev/docs/cookbook/networking/background-parsing)
