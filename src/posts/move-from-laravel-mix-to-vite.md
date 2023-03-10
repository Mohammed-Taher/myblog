---
layout: blog-post.njk
title: نقل مشروع Laravel من Laravel Mix إلى Vite
date: 2022-07-13
tags:
  - post
  - Laravel
---

أعلن فريق تطوير إطار العمل Laravel في الإصدار `9.19` أن إطار العمل سينتقل إلى استخدام [Vite](http://vitejs.dev) بدلًا عن [Laravel Mix](https://laravel-mix.com/) لتحزيم الأصول وتصريفها في مشاريع Laravel.

عملية تحزيم الأصول وتصريفها من العمليات الشائعة في مجال تطوير الواجهات الأمامية، والأصول هي ملفات CSS و Javascript وغيرها التي تستخدم في بناء الواجهة الأمامية. وعادة ما تكتب هذه الملفات باستخدام لغات وتقنيات لا تتعرف عليها المتصفحات، فعلى سبيل المثال يستخدم مطورو الواجهات الأمامية لغة Sass في كتابة ملفات CSS، إذ تضفي هذه اللغة -أي Sass- مزيدًا من القوة والمرونة إلى ملفات CSS، ولكن المشكلة أن المتصفحات لا تستطيع التعرف على هذه الملفات؛ لذا تستخدم برمجيات خاصة تدعى بالمصرفات تحوّل الشيفرة المكتوبة بلغة Sass إلى لغة CSS المعروفة من قبل المتصفح.

كذلك الأمر بالنسبة للغة Javascript؛ إذ يستخدم الكثير من المطورين لغة [Typescript](https://www.typescriptlang.org/) والتي لا يستطيع المتصفح أن يتعامل معها؛ ولكن باستخدام المصرفات يمكن تحويل الشيفرة إلى لغة Javascript.

تستخدم المصرّفات كذلك في تنفيذ عمليات أخرى مثل تحزيم الصور وتصغير الشيفرة وضغطها والكثير من العمليات المفيدة والتي تسهل عمل المبرمج وتوفّر عليه الوقت والجهد.

في الإصدارات السابقة من Laravel كان إطار العمل يستخدم مكتبة [Webpack](https://webpack.js.org/) لتحزيم الملفات، ويتيح التعامل مع هذه المكتبة باستخدام الحزمة Laravel Mix التي تقدّم عددًا من الدوال البسيطة والتي تساعد المبرمج على تنفيذ عمليات التصريف ونقل الملفات وتصغير الشيفرة وضغطها دون الحاجة إلى الدخول في تفاصيل هذه العمليات باستخدام Webpack.

أما الآن فقد انتقل إطار العمل Laravel في الإصدار 9.19 إلى استخدام Vite كبديل لحزمة Laravel Mix. طُوِّر Vite على يد [Evan You](https://twitter.com/youyuxi) وهو مطوّر إطار العمل Vue الشهير والغني عن التعريف. يمتاز Vite كما يدل الاسم (Vite يعني سريع باللغة الفرنسية) بسرعته الكبيرة في تحزيم الملفات، ويدعم Vite كذلك ميزة Hot Module Replacement وتعرف اختصارًا (HMR). والتي تتيح تحديث مكون معين في صفحة الويب بسرعة فائقة وبدقة عالية دون الحاجة إلى إعادة تحميل الصفحة أو تغيير حالة التطبيق.

## الانتقال إلى Vite

الخطوة الأولى في عملية الانتقال إلى Vite هي التأكد من تحديث إطار العمل Laravel إلى الإصدار `9.19.0`؛ لذا سننفذ الأمر التالي في المجلد الرئيسي للمشروع:

```shell
composer require laravel/framework:^9.19.0
```

بعد اكتمال هذه العملية، سنبدأ بتثبيت Vite و Laravel Vite Plugin باستخدام مدير الحزم `npm`:

```shell
npm install --save-dev vite laravel-vite-plugin
```

كذلك سنلغي تثبيت Laravel Mix و Webpack باستخدام الأمر التالي:

```shell
npm remove laravel-mix && rm webpack.mix.js
```

بعد اكتمال عملية إلغاء التثبيت، سننتقل إلى الملف `package.json` لتعديل القسم `scripts`. احذف كل شيء ضمن هذا القسم فقط، وأضف السطرين التاليين إليه:

```json
 "scripts": {
        "dev": "vite",
        "build": "vite build"
    },
```

الآن سنحتاج إلى تهيئة Vite للعمل؛ ولهذا سننشئ الملف `vite.config.js` في المجلد الرئيسي للمشروع. أضف الشيفرة التالية إلى هذا الملف:

```javascript
import laravel from "laravel-vite-plugin";
import { defineConfig } from "vite";

export default defineConfig({
  plugins: [laravel(["resources/css/app.css", "resources/js/app.js"])],
});
```

في البداية سنستورد الدالة `laravel` من الإضافة `laravel-vite-plugin`، وسنستورد كذلك الكائن `defineConfig` من المكتبة `vite`.

وفي الخاصية plugins سنستدعي الدالة `laravel` ونمرر إليها مصفوفة تتضمن الملفات التي نرغب في تصريفها باستخدام Vite. أضفنا هنا الملفين `app.css` و `app.js` ويمكنك بكل تأكيد إضافة المزيد من الملفات وفق الحاجة.

والآن لاستخدام الملفات التي سينتجها Vite بعد تنفيذ عملية التصريف سنستعين بالموجّه الجديد &#8206;`@vite` عوضًا عن الدالة `mix()`.

```php
@vite(['resources/css/app.css', 'resources/js/app.js'])
```

لاحظ أننا نمرر إلى الموجّه &#8206;`@vite` أسماء الملفات الأصلية غير المصرفة لا أسماء الملفات الناتجة عن عملية التصريف. ويمكنك هنا تمرير مصفوفة بأسماء الملفات وبهذا ستستخدم الموجه &#8206;`@vite` مرة واحدة، أو تمرير كل ملف على حدة واستخدام موجّه &#8206;`@vite` أكثر من مرة.

الخطوة الأخيرة هي تشغيل Vite:

```shell
npm run dev
```

ويمكنك تصريف الأصول في التطبيق لتكون جاهزة للنشر باستخدام الأمر:

```shell
npm run build
```

سيصرّف vite الأصول في المجلد `build` ضمن المجلد `public`؛ لذا يمكننا حذف المجلدين `js` و `css` من المجلد `public`.

يجدر التنبيه هنا إلى أن Vite يدعم وحدات ES فقط، وهذا يعني أنّك لا تستطيع استخدام `require` لاستيراد الملفات، بل يجب استخدام عبارة `import` لتنفيذ ذلك:

```javascript
import myPackage from "my-package";
```

## التعامل مع ملفات CSS

ينصح الدليل الرسمي للتحويل من Laravel Mix إلى Vite باستيراد ملفات CSS باستخدام ملفات Javscript للحصول على تجربة أفضل خصوصًا عند العمل على تطبيقات الصفحة الوحدة Single Page Applications:

```javascript
import "../css/app.css";
```

## ملاحظات بخصوص React و Vue

إن كنت تستخدم مكتبة React في بناء الواجهة الأمامية لتطبيقك، فيجب عليك أن تتأكد من أن جميع الملفات التي تحتوي على شيفرات JSX تنتهي باللاحقة &#8206;`.jsx` وليس &#8206;`.js`.

كذلك الأمر بالنسبة لإطار العمل Vue، إذ يجب إضافة اللاحقة &#8206;`.vue` إلى اسم الملف عند استيراد المكونات:

```javascript
import Button from "./Button.vue";
```

## إعادة تحميل الصفحات تلقائيًا

يقدّم Vite ميزة جميلة وهي إعادة تحميل الصفحة في المتفصح تلقائيًا Hot Reloading مع كل عملية تصريف للأصول، أي عند تنفيذ الأمر `npm run dev`.

هذه الميزة مقتصرة على ملفات الأصول فقط، بمعنى أنّ إجراء التعديلات على ملفات blade لن يؤدي إلى إعادة تحميل الصفحة تلقائيًا، ولكن تمكن المطوّر [Freek Van der Herten](https://twitter.com/freekmurze) من إيجاد حل لهذه المشكلة.

عدّل الملف `vite.config.js` ليصبح بالشكل التالي:

```javascript
import laravel from "laravel-vite-plugin";
import { defineConfig } from "vite";

export default defineConfig({
  plugins: [
    laravel(["resources/js/app.js"]),
    {
      name: "blade",
      handleHotUpdate({ file, server }) {
        if (file.endsWith(".blade.php")) {
          server.ws.send({
            type: "full-reload",
            path: "*",
          });
        }
      },
    },
  ],
});
```

والآن نفّذ الأمر `npm run dev`، وستلاحظ أن إجراء التعديلات على ملفات blade سيؤدي إلى إعادة تحميل الصفحة في المتصفح.

## المصادر

- [Upgrade Guide ](https://github.com/laravel/vite-plugin/blob/main/UPGRADE.md)
- [Moving A Laravel Webpack Project To Vite](https://christoph-rumpel.com/2022/6/moving-a-laravel-webpack-project-to-vite)
- [Using Laravel Vite to automatically refresh your browser when changing a Blade file ](https://freek.dev/2277-using-laravel-vite-to-automatically-refresh-your-browser-when-changing-a-blade-file)
