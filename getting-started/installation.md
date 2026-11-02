# التركيب

## 1) ضع المورد في السيرفر

انسخ مجلد `police-os` إلى `resources/` (مثلًا `resources/[police]/police-os`).

{% hint style="info" %}
اسم المجلد `police-os` هو المعتمد في هذا الدليل. إن غيّرته فاستعمل الاسم الجديد في `ensure` وفي رابط [واجهة HTTP API](../admin/http-api.md).
{% endhint %}

## 2) أضف آيتم التابلت

يُفتح النظام بآيتم قابل للاستخدام اسمه `Config.TabletItem` (الافتراضي `tablet`). أضفه إلى `qb-core/shared/items.lua`:

{% code title="qb-core/shared/items.lua" %}
```lua
tablet = { name = 'tablet', label = 'تابلت الشرطة', weight = 1000, type = 'item', image = 'tablet.png', unique = false, useable = true, shouldClose = true, description = 'تابلت نظام الشرطة' },
```
{% endcode %}

ثم ضع صورة `tablet.png` في مجلد صور المخزن الذي تستعمله.

{% hint style="warning" %}
المورد **لا يضيف الآيتم بنفسه**. بدون هذا السطر لن يجد الضابط ما يفتح به التابلت (يمكنه الفتح من بوت الأجنحة أو بالأمرين `/policetest` و`/createpolicetest`، انظر [التابلت وسطح المكتب](../usage/tablet.md)).
{% endhint %}

إن كنت تستعمل مخزنًا غير `qb-inventory` فعرّف الآيتم بالاسم نفسه وبصيغة مخزنك. يعتمد Police OS على `QBCore.Functions.CreateUseableItem`؛ فإن لم يدعمه مخزنك فاجعل استخدام الآيتم يُطلق الحدث التالي من جهة اللاعب، وهو يتحقق بنفسه أن اللاعب شرطي:

```lua
TriggerEvent('qb-policewings:client:OpenDesktop')
```

## 3) صورة الخلفية (اختياري)

ضع صورة الخلفية باسم **`back.png`** داخل `police-os/html/img/`. الحزمة لا تتضمّن هذه الصورة، وبدونها تظهر خلفية داكنة سادة ويعمل كل شيء آخر بشكل طبيعي.

{% hint style="info" %}
لا تحذف المجلد `html/img/apps/`: فيه أيقونات التطبيقات.
{% endhint %}

## 4) عدّل server.cfg

أضف الأسطر التالية **قبل** `ensure police-os`:

{% code title="server.cfg" %}
```ini
# أسرار Police OS: استعمل set وليس setr
set police_wh_results   "https://discord.com/api/webhooks/..."
set police_wh_accepted  "https://discord.com/api/webhooks/..."
set police_wh_rejected  "https://discord.com/api/webhooks/..."
set police_wh_mdt       "https://discord.com/api/webhooks/..."
set police_wh_security  "https://discord.com/api/webhooks/..."
set police_api_token    ""

ensure oxmysql
ensure qb-core
ensure qb-target
ensure qb-menu
ensure police-os
```
{% endcode %}

* كل ويبهوك اختياري: اتركه فارغًا فلا يُرسل شيء لذلك الغرض. شرح كل واحد في [ديسكورد والويبهوك](../configuration/discord.md).
* `police_api_token` فارغ يعني أن [واجهة HTTP API](../admin/http-api.md) معطّلة. ضع نصًا عشوائيًا طويلًا فقط إن أردت استعمالها.
* استعمل `set` وليس `setr`: الأمر `setr` يُرسل القيمة إلى أجهزة اللاعبين فتنكشف أسرارك.

## 5) اضبط الرتب

افتح `config.lua` وطابِق الرتب مع وظيفة الشرطة عندك قبل أول تشغيل: `AdminGrade` و`MinGradeForManager` و`BossMenu.MinGradeForBoss` و`TestSystem.MinGradeToManage`. تفاصيلها في [الرتب والوظائف](../configuration/ranks-and-jobs.md).

## 6) شغّل السيرفر

أعد تشغيل السيرفر، أو نفّذ `ensure police-os` من الكونسول. عند أول تشغيل يُنشئ المورد جداوله بنفسه ويطبع في الكونسول رسالة «قاعدة البيانات جاهزة». بعدها تابع [أول تشغيل](first-run.md) لإنشاء أول حساب.

## الترقية من نسخة أقدم

1. خذ نسخة احتياطية من المجلد القديم ومن قاعدة البيانات.
2. احذف مجلد المورد القديم وضع المجلد الجديد مكانه.
3. انقل تعديلاتك على `config.lua` إلى النسخة الجديدة **يدويًا**، فبعض المفاتيح مضافة حديثًا فلا تستبدل الملف الجديد بالقديم.
4. أعد صورة `html/img/back.png` إن كانت عندك.
5. احذف `server/check_permissions.lua` و`server/perm_commands.lua` إن بقيا من نسخة قديمة، فقد دُمجا في `permissions.lua`.
6. أعد التشغيل. الأعمدة الناقصة تُضاف تلقائيًا ويُطبع في الكونسول «أُضيف العمود ...» لكل عمود جديد.

كلمات المرور القديمة المخزّنة كنص صريح تتحوّل إلى تجزئة (Hash) تلقائيًا عند أول دخول ناجح لصاحبها.

## هيكل الملفات

```text
police-os/
├─ fxmanifest.lua
├─ config.lua
├─ locales/
│  ├─ locale.lua
│  └─ lang/  ar.lua  en.lua
├─ client/   main.lua  boss.lua
├─ server/   main.lua  boss.lua  accounts.lua  permissions.lua  login_system.lua
│            database.lua  security.lua  utils.lua  config_server.lua
│            police_discord_integration.lua  language_tool.lua
└─ html/     index.html  style.css  apps.css  script.js  i18n.js
             img/back.png   (تضيفه أنت)
             img/apps/*.png (أيقونات التطبيقات)
```

| الملف | وظيفته | تعدّله؟ |
|---|---|---|
| `config.lua` | كل إعدادات النظام | نعم |
| `locales/lang/*.lua` | ملفات اللغات | نعم (لتعديل الترجمة أو إضافة لغة) |
| `server/config_server.lua` | قراءة الأسرار من `server.cfg` وحدود محاولات الدخول | نادرًا |
| بقية ملفات `client` و`server` و`html` | منطق النظام وواجهته | لا حاجة |
