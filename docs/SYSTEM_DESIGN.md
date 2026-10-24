# سند جامع طراحی سامانه CEO Cockpit مپنا پارس
**شناسه:** ARC-MAPNA-GEN-2026-DESIGN | **نسخه:** 1.0.0 | **وضعیت:** مبنای طراحی با تصمیم‌های باز مشخص‌شده

**قاعده عامل توسعه:** هیچ KPI، مقدار، آستانه، نقش، API، اتصال، داده نمونه یا رفتار تأییدنشده‌ای را حدس نزن. موارد [نیازمند تأیید] باید نامشخص/غیرفعال بمانند. داده نمایشی همیشه DEMO است و نباید با REAL مخلوط شود.

## فهرست مطالب
1. خلاصه اجرایی
2. تحلیل نیازمندی‌ها
3. معماری سیستم
4. ساختار برنامه
5. طراحی داده
6. طراحی API
7. طراحی صفحات
8. سیستم طراحی UI/UX
9. یکپارچه‌سازی
10. امنیت
11. کارایی و قابلیت اطمینان
12. استقرار و عملیات
13. نقشه راه
14. پیوست‌ها و معیار پذیرش

## 1. خلاصه اجرایی
هدف، ساخت اتاق فرمان مدیریتی (CEO Cockpit) برای مشاهده قابل‌اتکای وضعیت بقا، نقدینگی و زنجیره ارزش مپنا/پارس ژنراتور است. سامانه لایه مشاهده و تصمیم‌یار است، نه جایگزین SAP یا سیستم عملیاتی.

### اصول منبع
- **Truth Over Beauty:** صحت، تازگی و شفافیت بر زیبایی مقدم است.
- **Fast Data:** ورود داده از طریق میکروفرم و قالب استاندارد Excel با حداقل اصطکاک.
- **Self-Hosted:** داده‌های حساس در زیرساخت داخلی بمانند و runtime به ابر عمومی وابسته نباشد.
- **مرحله‌بندی:** ورودی دستی/Excel ابتدا؛ SAP CO و SAP PP بعد از تصویب قرارداد داده و اعتبارسنجی.

### تصمیم معماری
پیشنهاد: React + TypeScript + Vite، Python + FastAPI، PostgreSQL، معماری ماژولار لایه‌ای (Modular Monolith)، استقرار On-Premise.
**دلیل:** عملیات ساده، مرزبندی روشن، هزینه کمتر از microservices و توسعه‌پذیری. جایگزین‌ها: microservices (پیچیدگی توزیع‌شده)، serverless (ناسازگاری احتمالی با خودمیزبانی)، monolith بدون مرزبندی (coupling).
**جمع‌بندی:** محصول لایه مدیریتی ردیابی‌پذیر است؛ KPI تعریف‌نشده نباید منتشر شود.

## 2. تحلیل نیازمندی‌ها

### 2.1 ذی‌نفعان
| ذی‌نفع | مسئولیت/نیاز | دسترسی پیشنهادی |
|---|---|---|
| مدیرعامل | مشاهده وضعیت کلان | مشاهده مدیریتی |
| مدیران حوزه‌ها | ارائه/تأیید داده حوزه | ثبت در حوزه مجاز |
| PMO | گردآوری و کنترل داده | ثبت، اعتبارسنجی، گزارش |
| مالک داده | تعریف معنا و منبع شاخص | تأیید داده مرتبط |
| مدیر سامانه | کاربران و تنظیمات | دسترسی مدیریتی محدود |
| IT/امنیت | استقرار، پایش، بازیابی | دسترسی زیرساختی |

### 2.2 نیازمندی‌های عملکردی
| ID | الزام | اولویت | معیار پذیرش |
|---|---|---|---|
| FR-001 | صفحه اصلی CEO Cockpit | Must | کارت‌ها منبع، زمان و کیفیت را نشان دهند |
| FR-002 | نمایش حوزه‌های پایش منبع | Must | عنوان/تعداد حوزه از فهرست مصوب؛ حوزه جدید ساخته نشود |
| FR-003 | شاخص مستقل سلامت داده | Must | تازگی، کامل‌بودن و اعتبار به‌وضوح مشخص باشند |
| FR-004 | میکروفرم ثبت داده | Must | اعتبارسنجی سمت سرور و ثبت مالک/زمان |
| FR-005 | واردسازی Excel | Must | preview، اعتبارسنجی ردیفی، گزارش خطا و تأیید |
| FR-006 | منشأ و تاریخ مؤثر | Must | هر رکورد معتبر منبع و زمان دارد |
| FR-007 | نمایش داده کهنه/ناقص/نامعتبر | Must | نامعلوم با سبز/عادی جایگزین نشود |
| FR-008 | RBAC | Must | مجوز سمت سرور بررسی شود |
| FR-009 | ممیزی تغییرات | Must | ثبت، اصلاح، تأیید، import و تنظیمات ممیزی شوند |
| FR-010 | جزئیات حوزه از کارت | Should | انتقال با حفظ فیلترها |
| FR-011 | اتصال SAP CO | Later | فقط پس از mapping و مجوز مصوب |
| FR-012 | اتصال SAP PP | Later | فقط پس از mapping و مجوز مصوب |
| FR-013 | خروجی گزارش | Should | فیلتر، زمان تولید و برچسب DEMO/REAL مشخص باشد |
| FR-014 | تنظیم سیاست تازگی | Must | فقط نقش مجاز؛ تغییر ممیزی شود |
| FR-015 | وضعیت اتصال منبع | Should | خطای منبع، داده قبلی را تازه جلوه ندهد |

### 2.3 نیازمندی‌های غیرعملکردی
| ID | الزام | آزمون/معیار |
|---|---|---|
| NFR-001 | Self-hosted | اجرا بدون وابستگی runtime به ابر عمومی |
| NFR-002 | TLS | HTTPS در مسیرهای شبکه |
| NFR-003 | رمزنگاری ذخیره‌سازی | دیسک و backup رمزگذاری‌شده، کلید مدیریت‌شده |
| NFR-004 | ردیابی | هر مقدار قابل ردیابی به منبع/زمان باشد |
| NFR-005 | جداسازی DEMO/REAL | جداسازی محیط و queryها |
| NFR-006 | دسترس‌پذیری | SLA/RTO/RPO [نیازمند تأیید] |
| NFR-007 | کارایی | هدف p95 پس از برآورد بار [نیازمند تأیید] |
| NFR-008 | Accessibility | صفحه‌کلید، focus، label و کنتراست |
| NFR-009 | RTL | فارسی و راست‌به‌چپ در همه صفحات |
| NFR-010 | نگهداشت‌پذیری | تست خودکار، migration و مستندات API |

### 2.4 Use Cases
| ID | مورد | جریان |
|---|---|---|
| UC-01 | مشاهده داشبورد | ورود ← داشبورد ← منبع/کیفیت/زمان |
| UC-02 | ثبت مقدار | انتخاب شاخص ← فرم ← validation ← ثبت |
| UC-03 | Excel | قالب ← بارگذاری ← preview ← اصلاح خطا ← تأیید |
| UC-04 | بررسی داده بد | هشدار ← جزئیات ← اصلاح/ارجاع به مالک |
| UC-05 | مدیریت کاربر | نقش مجاز ← تغییر ← audit |
| UC-06 | SAP | دریافت read-only ← اعتبارسنجی ← تطبیق ← انتشار |

### 2.5 شکاف‌ها و رفتار امن
| ID | شکاف | رفتار تا تأیید |
|---|---|---|
| GAP-001 | عنوان «پنج ستون» با چهار کارت و نوار مستقل سلامت داده ناسازگار است | چهار کارت مشاهده‌شده حفظ شوند؛ ستون پنجم ساخته نشود |
| GAP-002 | آستانه کهنگی/تعارض ۲۴ ساعت با ثبت هفتگی نامشخص | آستانه بدون مقدار پیش‌فرض؛ وضعیت «تعریف‌نشده» |
| GAP-003 | فرمول/واحد/منبع دقیق KPIها | KPI تا تعریف رسمی منتشر نشود |
| GAP-004 | نقش‌ها و تأییدکنندگان | نقش‌های پیشنهادی نیازمند تأیید |
| GAP-005 | حجم، کاربران، SLA، RTO/RPO | قبل از تولید تعیین شوند |
| GAP-006 | SSO/LDAP/روش ورود | با IT تصمیم‌گیری؛ روش محلی فقط در صورت تصویب |
| GAP-007 | API و مجوز SAP | اتصال غیرفعال تا تصویب |
| GAP-008 | نگهداری و حذف | حذف خودکار غیرفعال تا تصویب |

**جمع‌بندی:** نیازمندی‌های Must مبنای MVP هستند. اطلاعاتی که منبع تعریف نکرده، با عدد یا رفتار حدسی پر نمی‌شود.

## 3. معماری سیستم

### 3.1 معماری منطقی
    کاربر → React SPA/RTL → FastAPI REST → Application Services → Domain Validation → PostgreSQL
                                                   ├→ Excel Import → Private File Storage
                                                   ├→ Audit Service → PostgreSQL
                                                   └⇢ SAP CO Adapter (آینده، غیرفعال)
                                                    ⇢ SAP PP Adapter (آینده، غیرفعال)

### 3.2 نمودار مؤلفه Mermaid
    flowchart TB
      U[کاربر] --> WEB[React SPA]
      WEB --> API[FastAPI]
      API --> AUTH[Authentication and RBAC]
      API --> APP[Application Services]
      APP --> DOMAIN[Domain Rules]
      DOMAIN --> DB[(PostgreSQL)]
      APP --> XLS[Excel Import]
      XLS --> FS[Private Storage]
      APP --> AUDIT[Audit Service]
      APP -. future .-> CO[SAP CO Adapter]
      APP -. future .-> PP[SAP PP Adapter]

### 3.3 معماری فیزیکی
    Browser → Reverse Proxy/TLS → Static Frontend + FastAPI → PostgreSQL
                                                └→ Private File Storage
    PostgreSQL + File Storage → Encrypted Backup
    App + DB → Monitoring/Logs

**انتخاب:** Modular Monolith، لایه‌های Presentation/Application/Domain/Infrastructure و REST. صف پیام در MVP الزامی نیست. Microservices فقط با نیاز اندازه‌گیری‌شده بررسی شود.
**جریان داده:** ورود → اعتبارسنجی قالب/نوع → رد خطا یا ذخیره پیش‌نویس → کنترل دامنه/مالکیت → audit → انتشار معتبر → محاسبه کیفیت → نمایش.
**قواعد:** UI مستقیم به DB وصل نمی‌شود؛ قواعد در backend هستند؛ خطای SAP نباید timestamp داده را تازه کند؛ عملیات تغییر دارای actor، timestamp، request ID و نتیجه است.
**جمع‌بندی:** معماری کم‌پیچیدگی و قابل استقرار داخلی؛ اتصال‌های آینده در Adapter محصور می‌شوند.

## 4. ساختار برنامه

    MANG_01/
    ├── README.md
    ├── docs/
    │   ├── SYSTEM_DESIGN.md
    │   ├── API_CONTRACTS.md
    │   ├── DATA_DICTIONARY.md
    │   ├── SECURITY.md
    │   ├── OPERATIONS.md
    │   ├── DECISIONS.md
    │   └── ACCEPTANCE_TESTS.md
    ├── frontend/src/{app,pages,features,components,api,hooks,styles,tests}/
    ├── backend/app/{api/v1,core,domain,services,repositories,integrations/sap,imports,audit}/
    ├── backend/migrations/
    ├── backend/tests/
    ├── deploy/{Dockerfile.frontend,Dockerfile.backend,compose.yaml,reverse-proxy}/
    └── .github/workflows/

**الگوها:** Repository برای persistence؛ Service Layer برای use case؛ Adapter برای SAP/Excel؛ Strategy برای سیاست تازگی؛ DTO/Schema validation برای قرارداد؛ Dependency Injection برای تست؛ Audit Trail برای تاریخچه.
**پیکربندی:** environment/secret store؛ هیچ secret در Git؛ .env.example بدون اسرار؛ خطا با code پایدار و request_id؛ stack trace فقط در لاگ داخلی؛ لاگ ساخت‌یافته و ماسک اطلاعات حساس.
**جمع‌بندی:** منطق دامنه، API و UI جدا هستند و تغییرات باید با تست و تصمیم معماری ثبت شوند.

## 5. طراحی داده

### 5.1 پایگاه داده
PostgreSQL رابطه‌ای انتخاب می‌شود: ACID، constraint، روابط و migration. NoSQL در MVP توجیه نشده و فقط با نیاز اثبات‌شده قابل بررسی است.

### 5.2 ERD مفهومی
    erDiagram
      USER ||--o{ AUDIT_EVENT : performs
      ROLE ||--o{ USER_ROLE : assigned
      USER ||--o{ USER_ROLE : has
      DOMAIN_AREA ||--o{ KPI_DEFINITION : contains
      KPI_DEFINITION ||--o{ KPI_RECORD : records
      DATA_SOURCE ||--o{ KPI_RECORD : originates
      IMPORT_BATCH ||--o{ IMPORT_ROW : contains
      USER ||--o{ KPI_RECORD : submits

### 5.3 جداول
| جدول | فیلدهای اصلی | قاعده |
|---|---|---|
| users | id, username, display_name, status | username یکتا |
| roles | id, code, name | code یکتا |
| user_roles | user_id, role_id | ترکیب یکتا |
| domain_areas | id, code, name, status | حوزه فقط پس از تصویب فعال |
| kpi_definitions | id, code, name, unit, definition_status, freshness_policy_id | تعریف ناقص منتشر نشود |
| kpi_records | id, kpi_id, value, status, effective_at, observed_at, source_id, submitted_by | رکورد معتبر دارای منبع/تاریخ |
| data_sources | id, code, type, description | MANUAL/EXCEL/SAP/OTHER |
| import_batches | id, filename, status, uploaded_by, created_at | خصوصی و ممیزی‌شده |
| import_rows | id, batch_id, row_number, status, errors, normalized_payload | خطا به ردیف مرتبط |
| audit_events | id, actor_id, action, entity_type, entity_id, occurred_at, request_id, details | دسترسی محدود |
| freshness_policies | id, code, max_age, unit, approval_status | تا تصویب null و تعریف‌نشده |

ایندکس‌ها: users(username) unique؛ kpi_definitions(code) unique؛ kpi_records(kpi_definition_id,effective_at desc)؛ kpi_records(data_status,observed_at desc)؛ audit_events(entity_type,entity_id,occurred_at desc)؛ import_rows(batch_id,row_number) unique. از حذف فیزیکی رکورد ممیزی‌شده پرهیز شود.
**مهاجرت/بازیابی:** migration نسخه‌بندی‌شده؛ backup رمزگذاری‌شده و جدا از میزبان؛ restore دوره‌ای آزمایش شود. RPO/RTO، نگهداری و سقف فایل [نیازمند تأیید]. حذف خودکار خاموش. واردسازی تکراری باید idempotent یا با batch ID کنترل شود.
**جمع‌بندی:** هر رکورد مقدار، وضعیت، منبع، زمان و مالک دارد؛ KPI بدون تعریف مصوب منتشر نمی‌شود.

## 6. طراحی API

Base path: /api/v1؛ JSON UTF-8؛ auth برای مسیرهای غیرعمومی؛ pagination؛ خطای یکنواخت با code/message/details/request_id.

| Method/Path | کاربرد | وضعیت‌ها |
|---|---|---|
| GET /health | سلامت | 200, 503 |
| GET /dashboard/summary | کارت‌ها و کیفیت | 200, 401, 403 |
| GET /areas | حوزه‌ها | 200, 401 |
| GET /kpis | فهرست و فیلتر | 200, 400 |
| GET /kpis/{id} | تعریف شاخص | 200, 404 |
| GET /kpis/{id}/records | تاریخچه | 200, 404 |
| POST /kpis/{id}/records | ثبت رکورد | 201, 409, 422 |
| PATCH /records/{id} | اصلاح مجاز | 200, 403, 404, 422 |
| POST /imports | بارگذاری | 202, 413, 415, 422 |
| GET /imports/{id} | وضعیت batch | 200, 404 |
| GET /imports/{id}/errors | خطاهای ردیفی | 200, 404 |
| POST /imports/{id}/commit | تأیید import | 200, 403, 409 |
| GET /admin/users | فهرست کاربران | 200, 403 |
| PUT /admin/users/{id}/roles | تغییر نقش | 200, 403, 422 |
| GET /audit | ممیزی | 200, 403 |
| GET /integrations/status | سلامت اتصال‌ها | 200, 403 |

نمونه OpenAPI خلاصه:
    openapi: 3.1.0
    info:
      title: MANG_01 CEO Cockpit API
      version: 1.0.0
    servers:
      - url: /api/v1
    paths:
      /dashboard/summary:
        get:
          security:
            - bearerAuth: []
          responses:
            "200": { description: Summary with source, freshness and quality }
            "401": { description: Authentication required }
            "403": { description: Insufficient permission }
    components:
      securitySchemes:
        bearerAuth:
          type: http
          scheme: bearer
          bearerFormat: JWT

نسخه‌بندی: تغییر سازگار در همان نسخه؛ تغییر ناسازگار با نسخه جدید. قراردادها تست contract داشته باشند. فایل با allowlist نوع، بررسی محتوای واقعی و محدودیت حجم مصوب پذیرفته شود.
**جمع‌بندی:** API مستند، نسخه‌بندی‌شده و مستقل از UI باشد؛ endpoint SAP تا تصویب قرارداد فعال نشود.

## 7. طراحی صفحات

### قواعد مشترک
فارسی RTL؛ هر مقدار دارای واحد، منبع، زمان و کیفیت؛ کلیک کارت به جزئیات همان حوزه؛ حالت‌های Loading/Empty/Error/Success/Partial؛ داده ناموجود با صفر یا سبز جایگزین نشود؛ مجوز همیشه سمت سرور بررسی شود.

### 7.1 ورود
هدف: ورود امن. عناصر: شناسه/روش SSO مصوب، گذرواژه در صورت نیاز، پیام خطای عمومی. حالت‌ها: loading، ورود موفق، نامعتبر، حساب غیرفعال، سرویس قطع. SSO/LDAP نیازمند تصمیم IT.

### 7.2 CEO Cockpit
کاربران: مدیرعامل/مدیران مجاز. ترتیب:
1. هدر: نام، محیط DEMO/REAL، زمان آخرین به‌روزرسانی، سلامت سرویس و کاربر.
2. نوار سلامت داده با فرمول مصوب.
3. کارت‌های حوزه‌های عملیاتی مصوب؛ تعداد نهایی نیازمند تأیید است.
4. موارد نیازمند توجه فقط بر اساس قواعد مصوب.
5. وضعیت منابع و آخرین دریافت موفق/ناموفق.
هر کارت: عنوان، مقدار/واحد مصوب، روند فقط با تاریخچه کافی، کیفیت، تاریخ مؤثر، منبع، لینک جزئیات. ممنوع: KPI/آستانه ساختگی یا مقایسه بدون دوره. حالت‌ها: skeleton، داده موجود نیست، خطا با request ID، داده جزئی با هشدار، موفق با timestamp.

### 7.3 جزئیات حوزه
فیلتر زمان/منبع/وضعیت؛ جدول رکوردها با زمان مؤثر/مشاهده، مالک، منبع و اعتبار؛ ثبت فقط با مجوز. بازگشت فیلترها را حفظ کند.

### 7.4 تعریف شاخص‌ها
کد، عنوان، تعریف، واحد، منبع مجاز، مالک، فرمول، سیاست تازگی و وضعیت Draft/Approved/Inactive. انتشار بدون تعریف/واحد/مالک/منبع ممنوع.

### 7.5 میکروفرم
انتخاب شاخص مجاز، مقدار، واحد از تعریف، تاریخ مؤثر، منبع و یادداشت اختیاری. اعتبارسنجی backend، تاریخ، دسترسی و تکراری‌بودن. موفقیت شناسه و زمان؛ خطا فیلد مشخص و حفظ ورودی.

### 7.6 واردسازی Excel
قالب نسخه‌دار → انتخاب فایل → بررسی → preview → خطایابی → تأیید → نتیجه. ستون ناشناخته بی‌صدا حذف نشود؛ ردیف نامعتبر منتشر نشود؛ batch و نتیجه ممیزی شود؛ تکرار کنترل شود. حالت‌های فایل نامعتبر/بزرگ/قالب ناسازگار/خطاهای ردیفی/موفقیت. واردسازی جزئی فقط پس از تصویب سیاست.

### 7.7 سلامت داده
فهرست کهنه، ناقص، نامعتبر، بدون مالک/منبع، تعریف‌نشده و منبع قطع‌شده. هر وضعیت تعریف، شمارش، زمان محاسبه و لینک رکورد داشته باشد. آستانه تازگی تا تصویب بدون مقدار پیش‌فرض.

### 7.8 ممیزی
actor، زمان، عمل، موجودیت، شناسه و request ID؛ فیلتر کاربر/زمان/عمل؛ رویدادها از UI قابل ویرایش/حذف نباشند؛ جزئیات حساس ماسک شوند.

### 7.9 کاربران و نقش‌ها
فهرست، وضعیت، نقش و تغییرات؛ حداقل دسترسی؛ تغییر نقش audit شود؛ کاربر دارای تاریخچه حذف فیزیکی نشود.

### 7.10 اتصال‌ها و تنظیمات
منبع، آخرین تلاش/موفقیت، وضعیت و خلاصه خطا؛ secrets هرگز نمایش داده نشوند؛ SAP تا تصویب غیرفعال؛ تغییر پارامتر audit شود.

### ناوبری
    flowchart TD
      LOGIN[ورود] --> DASH[CEO Cockpit]
      DASH --> AREA[جزئیات حوزه]
      DASH --> QUALITY[سلامت داده]
      AREA --> ENTRY[ثبت داده]
      DASH --> IMPORT[واردسازی Excel]
      DASH --> DEFINITIONS[تعریف شاخص]
      DASH --> INTEGRATIONS[اتصال‌ها]
      ADMIN[منوی مدیریت با مجوز] --> USERS[کاربران]
      ADMIN --> AUDIT[ممیزی]
      ADMIN --> CONFIG[تنظیمات]

Responsive: دسکتاپ اولویت؛ موبایل کارت تک‌ستونه و جدول قابل بازشدن. هیچ تعاملی صرفاً hover نباشد. WCAG 2.2 AA به‌عنوان هدف طراحی؛ کنتراست، focus، صفحه‌کلید و برچسب فرم آزموده شود.
**جمع‌بندی:** صفحه اول وضعیت را آشکار می‌کند؛ جزئیات، منشأ و کیفیت باید قابل بررسی باشند.

## 8. سیستم طراحی UI/UX

فلسفه: مدیریتی، آرام، کم‌ازدحام، RTL واقعی و مبتنی بر شواهد. پالت زیر پیشنهاد طراحی است، نه هویت بصری مصوب.

| Token | HEX | کاربرد |
|---|---|---|
| bg | #F5F7FA | پس‌زمینه روشن |
| surface | #FFFFFF | کارت |
| text | #172B4D | متن اصلی |
| muted | #526581 | متن ثانویه |
| border | #D7DEE8 | مرز |
| primary | #174A7E | اقدام اصلی |
| info | #176B87 | اطلاع |
| success | #237A57 | معتبر/موفق فقط با شواهد |
| warning | #946200 | نیازمند بررسی |
| danger | #B42318 | خطا/نامعتبر |
| unknown | #667085 | نامعلوم |
| dark-bg | #101828 | پس‌زمینه تاریک |
| dark-surface | #1D2939 | پنل تاریک |
| dark-text | #F2F4F7 | متن تاریک |

تایپوگرافی پیشنهادی: Vazirmatn محلی با fallback به Tahoma/sans-serif؛ متن 14–16px، جدول حداقل 13px، عنوان صفحه 24–28px، کارت 16–18px. استفاده از فونت باید از منبع مجاز سازمان باشد.
Spacing: مقیاس 4px: 4,8,12,16,24,32,40,48؛ radius کارت 10–12px؛ گرید دسکتاپ 12 ستون و gap 16–24px.
مؤلفه‌ها: Button (primary/secondary/danger/loading/disabled)، StatusBadge، KPI Card، DataTable، FormField، FileDropzone، Toast، Modal، AuditTimeline، Chart. نمودار فقط با داده کافی، محور و واحد مشخص.
Dark/Light: روشن پیش‌فرض پیشنهادی؛ هر دو حالت با کنتراست یکسان؛ Dark اختیاری. Motion 120–200ms، بدون انیمیشن دائم، احترام به reduced-motion.
**جمع‌بندی:** tokenها و اجزا مشترک‌اند؛ رنگ و فونت قبل از تثبیت باید تأیید شوند.

## 9. یکپارچه‌سازی و وابستگی‌ها

    flowchart LR
      UI[Frontend] --> API[Backend API]
      API --> DOMAIN[Domain Services]
      DOMAIN --> DB[(PostgreSQL)]
      DOMAIN --> AUDIT[Audit]
      API --> XLS[Excel Import]
      XLS --> FILES[Private Storage]
      DOMAIN -. future .-> CO[SAP CO Adapter]
      DOMAIN -. future .-> PP[SAP PP Adapter]
      API --> IDP[Identity Provider if approved]

| وابستگی | MVP | ریسک | کنترل |
|---|---|---|---|
| PostgreSQL | بله | خرابی/از دست‌رفتن | backup و restore test |
| Storage خصوصی | برای Excel | افشای/فایل مخرب | دسترسی خصوصی، allowlist و محدودیت |
| Identity Provider | تصمیم‌پذیر | قطعی/پیکربندی | تست و روش جایگزین مصوب |
| SAP CO | خیر | mapping/مجوز | adapter و read-only ابتدا |
| SAP PP | خیر | معنای وضعیت تولید | mapping با مالک داده |
| Reverse proxy/TLS | تولید بله | پیکربندی ناامن | hardening |
| Monitoring | تولید بله | نقاط کور | health/alerts |

اتصال SAP فقط پس از تصویب منبع رسمی، فیلدها، واحد، timezone، تناوب، مالک و سیاست خطا. شروع read-only؛ نوشتن به SAP خارج از دامنه است. هیچ جدول یا endpoint SAP بدون بررسی landscape فرض نشود.
**جمع‌بندی:** فقط وابستگی‌های لازم MVP فعال؛ SAP مرحله‌ای و کنترل‌شده.

## 10. امنیت و حریم خصوصی

- RBAC با حداقل دسترسی و کنترل مجوز برای هر عملیات در backend.
- حساب مدیریتی محدود و ممیزی‌شده؛ MFA/session timeout/password policy بر اساس استاندارد سازمان [نیازمند تأیید].
- TLS؛ رمزنگاری دیسک و backup؛ secrets در secret manager/environment امن؛ هرگز در Git یا log.
- فایل‌ها خارج از public web root؛ allowlist، بررسی MIME/content، محدودیت حجم و ضدویروس در صورت وجود ابزار مصوب.
- Injection: parameterized queries/ORM؛ XSS: escaping/CSP؛ CSRF برای cookie auth؛ SSRF: allowlist؛ brute force: rate limit؛ supply chain: lockfile و dependency scan؛ خطاها بدون stack trace به کاربر.
- Audit محدود و تا حد امکان append-only؛ لاگ‌ها ماسک‌شده.
- GDPR و سایر قوانین فقط پس از ارزیابی حقوقی و تعیین شمول؛ این سند ادعای انطباق حقوقی نمی‌کند.
**جمع‌بندی:** امنیت در احراز هویت، مجوز، فایل، لاگ و استقرار اعمال شود و با تست اثبات گردد.

## 11. کارایی و قابلیت اطمینان

- MVP با یک backend و PostgreSQL؛ cache فقط با timestamp و invalidation پس از ثبت.
- Pagination و query محدود؛ index براساس query واقعی و EXPLAIN ANALYZE.
- ابتدا مقیاس عمودی؛ مقیاس افقی پس از حذف state محلی و آزمون concurrency.
- Load balancer در صورت چند replica.
- Metrics: latency، error rate، requests، DB connections، زمان import، حجم فایل و آخرین موفقیت منبع.
- Logs ساخت‌یافته با request ID؛ liveness/readiness جدا.
- هشدار برای خطاهای تکراری، فضای کم، backup ناموفق، قطع منبع و خطاهای import؛ threshold [نیازمند تأیید عملیات].
- SLA/RTO/RPO قبل از Go-Live تعیین شود. بعد از قطعی منبع، آخرین مقدار با برچسب آخرین دریافت موفق/قدیمی نمایش داده شود.
**جمع‌بندی:** اندازه‌گیری و بازیابی مقدم بر بهینه‌سازی حدسی است.

## 12. استقرار و عملیات

استقرار On-Premise/private network: reverse proxy با TLS، frontend static، FastAPI، PostgreSQL و storage خصوصی. Docker برای تکرارپذیری؛ Kubernetes اختیاری بر اساس استاندارد IT. runtime باید بدون اینترنت عمومی کار کند مگر وابستگی مصوب.

محیط‌ها:
- DEV: داده ساختگی با برچسب DEMO.
- STAGING: DEMO یا داده مجاز ناشناس‌شده.
- PROD: فقط داده واقعی مجاز، DB و secrets جدا.
- کپی داده واقعی به DEV بدون مجوز/ناشناس‌سازی ممنوع.

Pipeline:
    Commit/PR → Lint/Type Check → Unit/Integration/Contract Tests
    → Dependency/Secret Scan → Immutable Build → Staging
    → Acceptance Tests → Approval → Production → Health Check/Rollback Gate

Migration با backup و طرح rollback؛ ثبت commit و نسخه image؛ secret در CI log ممنوع؛ runbook برای بازیابی، DB، دیسک، import و SAP.
**جمع‌بندی:** انتشار تکرارپذیر و قابل بازگشت؛ Go-Live بدون UAT، backup قابل بازیابی و تأیید امنیت/عملیات ممنوع.

## 13. نقشه راه

| فاز | خروجی | معیار عبور |
|---|---|---|
| P0 | تصمیم حوزه/KPI/نقش/آستانه/NFR | تصویب تصمیم‌های باز بحرانی |
| P1 | اسکلت، auth، DB, migrations, logs, CI | تست پایه و build تکرارپذیر |
| P2 | داشبورد، کارت، کیفیت و جزئیات | هر مقدار قابل ردیابی |
| P3 | میکروفرم، Excel، preview، validation، audit | تست معتبر/نامعتبر/تکراری |
| P4 | admin، monitoring، backup/restore، runbook | تست مجوز و بازیابی |
| P5 | SAP CO/PP read-only و mapping | تأیید مالک SAP و داده |
| P6 | بار، امنیت، UAT و آموزش | امضای Go-Live |

مدت فازها تا تعیین تیم، دامنه داده و زیرساخت تخمین زده نمی‌شود.

MoSCoW:
- **Must:** داشبورد، کیفیت/تازگی، ورود دستی، Excel، منبع/زمان، RBAC، audit، self-hosted، جداسازی DEMO/REAL.
- **Should:** فیلتر/گزارش، وضعیت اتصال، جست‌وجوی ممیزی.
- **Could:** Dark mode و نمودار پیشرفته.
- **Won't in MVP:** نوشتن SAP، تصمیم خودکار بدون تأیید مدیر، ابر عمومی اجباری، KPI/آستانه ساختگی.

ریسک‌ها: KPI نامشخص → عدم انتشار؛ داده دیرهنگام → timestamp/سیاست؛ Excel خراب → preview/validation؛ مالکیت متعارض → تعیین owner/approver؛ SAP زودهنگام → فاز read-only؛ DEMO/REAL مخلوط → جداسازی محیط؛ backup نامعتبر → restore test.

تیم: Product Owner، مالکان داده، معمار/Tech Lead، frontend، backend، QA automation، DevOps، امنیت و متخصص SAP برای فاز اتصال. مسئولیت‌ها حتی در تیم کوچک صریح باشند.
**جمع‌بندی:** توسعه براساس معیار عبور است؛ زمان‌بندی بدون داده ورودی حدس زده نمی‌شود.

## 14. پیوست‌ها و معیار پذیرش

### 14.1 واژه‌نامه
- CEO Cockpit: اتاق فرمان مدیریتی.
- KPI: شاخص با تعریف و واحد مصوب.
- Data Freshness: تازگی نسبت به سیاست مصوب.
- Data Quality: کامل‌بودن، اعتبار، منشأ و تازگی.
- Microform: فرم کوتاه ثبت داده.
- Self-hosted: زیرساخت تحت کنترل سازمان.
- RBAC: کنترل دسترسی مبتنی بر نقش.
- Audit Trail: تاریخچه قابل ردیابی عملیات.
- DEMO: داده فرضی/نمایشی.
- Adapter: مرز فنی اتصال بیرونی.

### 14.2 تصمیم‌های اجباری پیش از تولید
1. رفع ابهام پنج ستون در برابر چهار کارت و نوار سلامت داده.
2. تصویب حوزه‌ها، KPIها، فرمول، واحد، مالک و منبع.
3. تصویب آستانه تازگی و رفتار داده ناموجود.
4. تصویب نقش‌ها، approval و روش احراز هویت.
5. تصویب SLA، RTO/RPO، حجم کاربران/داده و سقف فایل.
6. تصویب نگهداری/حذف و الزامات حقوقی.
7. تصویب mapping و مجوز SAP CO/PP.
8. تصویب هویت بصری و فونت مجاز.

### 14.3 چک‌لیست پذیرش
- [ ] هر مقدار داشبورد منبع، زمان و کیفیت دارد.
- [ ] داده تعریف‌نشده/کهنه به‌صورت سبز/عادی نمایش داده نمی‌شود.
- [ ] KPI یا مقدار واقعی ساختگی در PROD وجود ندارد.
- [ ] DEMO و REAL جدا و قابل تشخیص‌اند.
- [ ] اعتبارسنجی فرم/Excel در backend انجام می‌شود.
- [ ] فایل نامعتبر بدون انتشار رد می‌شود و خطاها گزارش می‌شوند.
- [ ] تغییرات مهم audit می‌شوند.
- [ ] API برای کاربر غیرمجاز عملیات ممنوع را رد می‌کند.
- [ ] صفحات حالت Loading/Empty/Error/Success دارند.
- [ ] RTL، keyboard و contrast آزموده شده‌اند.
- [ ] backup خارج از میزبان و restore آزمایش شده است.
- [ ] API قرارداد مستند و تست خودکار دارد.
- [ ] Go-Live پس از UAT و تأیید امنیت/عملیات است.

### 14.4 دستور صریح برای عامل توسعه
1. ابتدا این سند و همه docs را بخوان.
2. موارد 14.2 را به decision log با مالک، وضعیت و اثر تبدیل کن.
3. اگر تصمیم باز رفتار سیستم را تغییر می‌دهد، مقدار/رفتار فرضی نساز؛ placeholder «نیازمند تأیید» بگذار و قابلیت وابسته را غیرفعال کن.
4. هر FR/NFR را به تست قابل اجرا نگاشت کن.
5. قراردادهای داده/API را پیش از UI پیاده‌سازی کن.
6. مورد خارج از سند را بدون decision ثبت‌شده اضافه نکن.
7. تصمیم جدید را در docs/DECISIONS.md با گزینه‌ها، دلیل، اثر و تأییدکننده ثبت کن.
8. تست‌ها و موارد پوشش‌نیافته را در docs/ACCEPTANCE_TESTS.md ثبت کن.
9. seed داده کسب‌وکاری در PROD ممنوع؛ داده نمونه فقط DEMO.
10. تکمیل توسعه به معنی قبولی نیست؛ معیارها باید با تست و شواهد پاس شوند.

**وضعیت:** آماده فاز P0/P1؛ KPI واقعی و Go-Live وابسته به تصویب تصمیم‌های باز است.
