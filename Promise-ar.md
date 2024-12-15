---
title: Promise
slug: Web/JavaScript/Reference/Global_Objects/Promise
page-type: javascript-class
browser-compat: javascript.builtins.Promise
---

{{JSRef}}

يمثل كائن **`Promise`** الإكمال المحتمل (أو الفشل) لعملية غير متزامنة ونتيجتها.

لفهم كيفية عمل الـ promises وكيف يمكنك استخدامها، ننصحك بقراءة [استخدام Promises](/en-US/docs/Web/JavaScript/Guide/Using_promises) أولاً.

## الوصف

الـ `Promise` هو وسيط لقيمة غير معروفة بالضرورة عند إنشاء الـ promise. يتيح لك ربط معالجات مع النجاح المحتمل لعملية غير متزامنة أو سبب فشلها. هذا يسمح للطرق غير المتزامنة بإرجاع قيم مثل الطرق المتزامنة: بدلاً من إرجاع القيمة النهائية مباشرة، تقوم الطريقة غير المتزامنة بإرجاع _promise_ لتوفير القيمة في وقت ما في المستقبل.

يكون الـ `Promise` في إحدى هذه الحالات:

- _pending_: الحالة الأولية، لم يتم الوفاء بها أو رفضها.
- _fulfilled_: تعني أن العملية اكتملت بنجاح.
- _rejected_: تعني أن العملية فشلت.

الحالة _النهائية_ لـ promise معلق يمكن أن تكون إما _fulfilled_ مع قيمة أو _rejected_ مع سبب (خطأ).
عندما يحدث أي من هذه الخيارات، يتم استدعاء المعالجات المرتبطة المضافة إلى قائمة الانتظار بواسطة طريقة `then` الخاصة بالـ promise. إذا تم الوفاء بالـ promise أو رفضه بالفعل عند إرفاق معالج مقابل، سيتم استدعاء المعالج، لذلك لا يوجد تنافس بين إكمال العملية غير المتزامنة وإرفاق معالجاتها.

يقال إن الـ promise قد _تم تسويته_ إذا كان إما تم الوفاء به أو رفضه، ولكن ليس معلقاً.

![مخطط انسيابي يوضح كيف تنتقل حالة Promise بين pending و fulfilled و rejected عبر معالجات then/catch. يمكن للـ promise المعلق أن يصبح إما fulfilled أو rejected. إذا تم الوفاء به، يتم تنفيذ معالج "عند الوفاء"، أو المعامل الأول لطريقة then()، وينفذ المزيد من الإجراءات غير المتزامنة. إذا تم رفضه، يتم تنفيذ معالج الخطأ، إما الممرر كمعامل ثانٍ لطريقة then() أو كمعامل وحيد لطريقة catch().](promises.png)

ستسمع أيضاً مصطلح _resolved_ يستخدم مع الـ promises - وهذا يعني أن الـ promise قد تم تسويته أو "تثبيته" ليتطابق مع الحالة النهائية لـ promise آخر، وأن المزيد من الحل أو الرفض ليس له تأثير. يحتوي مستند [States and fates](https://github.com/domenic/promises-unwrapping/blob/master/docs/states-and-fates.md) من اقتراح Promise الأصلي على مزيد من التفاصيل حول مصطلحات Promise. في اللغة العامية، غالباً ما تكون الـ promises "المحلولة" مكافئة للـ promises "المُوفى بها"، ولكن كما هو موضح في "States and fates"، يمكن أن تكون الـ promises المحلولة معلقة أو مرفوضة أيضاً. على سبيل المثال:

```js
new Promise((resolveOuter) => {
  resolveOuter(
    new Promise((resolveInner) => {
      setTimeout(resolveInner, 1000);
    }),
  );
});
```

هذا الـ promise تم _حله_ بالفعل في وقت إنشائه (لأن `resolveOuter` يتم استدعاؤه بشكل متزامن)، لكنه تم حله بـ promise آخر، وبالتالي لن يتم _الوفاء_ به حتى مرور ثانية واحدة، عندما يتم الوفاء بالـ promise الداخلي. في الممارسة العملية، غالباً ما يتم "الحل" خلف الكواليس وغير قابل للملاحظة، وفقط الوفاء أو الرفض هو المرئي.

> [!NOTE]
> تمتلك العديد من اللغات الأخرى آليات للتقييم الكسول وتأجيل الحساب، والتي تسميها أيضاً "promises"، مثل Scheme. تمثل Promises في JavaScript العمليات التي تحدث بالفعل، والتي يمكن ربطها بدوال callback. إذا كنت تبحث عن تقييم تعبير بشكل كسول، فكر في استخدام دالة بدون معاملات مثل `f = () => expression` لإنشاء التعبير المُقيَّم بكسل، و `f()` لتقييم التعبير فوراً.

لا يمتلك `Promise` نفسه بروتوكولاً من الدرجة الأولى للإلغاء، ولكن قد تتمكن من إلغاء العملية غير المتزامنة الأساسية مباشرة، عادةً باستخدام [`AbortController`](/en-US/docs/Web/API/AbortController).

### Promises المتسلسلة

تُستخدم طرق {{jsxref("Promise/then", "then()")}} و {{jsxref("Promise/catch", "catch()")}} و {{jsxref("Promise/finally", "finally()")}} لربط إجراء إضافي مع promise تم تسويته. طريقة `then()` تأخذ ما يصل إلى معاملين؛ المعامل الأول هو دالة callback لحالة الوفاء بالـ promise، والمعامل الثاني هو دالة callback لحالة الرفض. طريقتا `catch()` و `finally()` تستدعيان `then()` داخلياً وتجعلان معالجة الأخطاء أقل تطويلاً. على سبيل المثال، `catch()` هو في الواقع مجرد `then()` بدون تمرير معالج الوفاء. نظراً لأن هذه الطرق تُرجع promises، يمكن تسلسلها. على سبيل المثال:

```js
const myPromise = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("foo");
  }, 300);
});

myPromise
  .then(handleFulfilledA, handleRejectedA)
  .then(handleFulfilledB, handleRejectedB)
  .then(handleFulfilledC, handleRejectedC);
```

سنستخدم المصطلحات التالية: _الـ promise الأولي_ هو الـ promise الذي يتم استدعاء `then` عليه؛ _الـ promise الجديد_ هو الـ promise الذي يُرجعه `then`. المعالجان اللذان يتم تمريرهما إلى `then` يُسميان _معالج الوفاء_ و _معالج الرفض_، على التوالي.

حالة الـ promise الأولي المُسوَّى تحدد أي معالج سيتم تنفيذه.

- إذا تم الوفاء بالـ promise الأولي، يتم استدعاء معالج الوفاء مع قيمة الوفاء.
- إذا تم رفض الـ promise الأولي، يتم استدعاء معالج الرفض مع سبب الرفض.

إكمال المعالج يحدد الحالة المُسوَّاة للـ promise الجديد.

- إذا أرجع المعالج قيمة [thenable](#thenables)، يتم تسوية الـ promise الجديد بنفس حالة القيمة المُرجعة.
- إذا أرجع المعالج قيمة غير thenable، يتم الوفاء بالـ promise الجديد مع القيمة المُرجعة.
- إذا رمى المعالج خطأً، يتم رفض الـ promise الجديد مع الخطأ المرمي.
- إذا لم يكن للـ promise الأولي معالج مقابل مرفق، سيتم تسوية الـ promise الجديد بنفس حالة الـ promise الأولي - أي أنه بدون معالج رفض، يظل الـ promise المرفوض مرفوضاً بنفس السبب.

على سبيل المثال، في الكود أعلاه، إذا تم رفض `myPromise`، سيتم استدعاء `handleRejectedA`، وإذا أكمل `handleRejectedA` بشكل طبيعي (بدون رمي أو إرجاع promise مرفوض)، سيتم الوفاء بالـ promise المُرجع من `then` الأول بدلاً من البقاء مرفوضاً. لذلك، إذا كان يجب معالجة الخطأ فوراً، ولكننا نريد الحفاظ على حالة الخطأ في السلسلة، يجب علينا رمي خطأ من نوع ما في معالج الرفض. من ناحية أخرى، في غياب حاجة فورية، يمكننا ترك معالجة الأخطاء حتى معالج `catch()` النهائي.

```js
myPromise
  .then(handleFulfilledA)
  .then(handleFulfilledB)
  .then(handleFulfilledC)
  .catch(handleRejectedAny);
```

باستخدام [دوال السهم](/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions) لدوال callback، قد يبدو تنفيذ سلسلة الـ promise كما يلي:

```js
myPromise
  .then((value) => `${value} and bar`)
  .then((value) => `${value} and bar again`)
  .then((value) => `${value} and again`)
  .then((value) => `${value} and again`)
  .then((value) => {
    console.log(value);
  })
  .catch((err) => {
    console.error(err);
  });
```

> [!NOTE]
> للتنفيذ الأسرع، يجب تنفيذ جميع الإجراءات المتزامنة في معالج واحد، وإلا سيستغرق الأمر عدة نقرات لتنفيذ جميع المعالجات بالتسلسل.

يحتفظ JavaScript بـ [قائمة مهام](/en-US/docs/Web/JavaScript/Event_loop). في كل مرة، يختار JavaScript مهمة من القائمة وينفذها حتى الاكتمال. يتم تحديد المهام بواسطة منفذ منشئ `Promise()`، والمعالجات الممررة إلى `then`، أو أي واجهة برمجة تطبيقات للمنصة تُرجع promise. تمثل الـ promises في السلسلة علاقة التبعية بين هذه المهام. عندما يتم تسوية promise، تتم إضافة المعالجات المرتبطة به إلى نهاية قائمة المهام.

يمكن لـ promise أن يشارك في أكثر من سلسلة واحدة. بالنسبة للكود التالي، سيؤدي الوفاء بـ `promiseA` إلى إضافة كل من `handleFulfilled1` و `handleFulfilled2` إلى قائمة المهام. نظراً لأن `handleFulfilled1` تم تسجيله أولاً، سيتم استدعاؤه أولاً.

```js
const promiseA = new Promise(myExecutorFunc);
const promiseB = promiseA.then(handleFulfilled1, handleRejected1);
const promiseC = promiseA.then(handleFulfilled2, handleRejected2);
```

يمكن تعيين إجراء لـ promise تم تسويته بالفعل. في هذه الحالة، تتم إضافة الإجراء فوراً إلى نهاية قائمة المهام وسيتم تنفيذه عند اكتمال جميع المهام الموجودة. لذلك، سيحدث الإجراء لـ promise "مُسوَّى" بالفعل فقط بعد اكتمال الكود المتزامن الحالي ومرور نقرة واحدة على الأقل في الحلقة. هذا يضمن أن إجراءات الـ promise غير متزامنة.

```js
const promiseA = new Promise((resolve, reject) => {
  resolve(777);
});
// في هذه النقطة، "promiseA" تم تسويته بالفعل
promiseA.then((val) => console.log("تسجيل غير متزامن له القيمة:", val));
console.log("تسجيل فوري");

// ينتج الإخراج بهذا الترتيب:
// تسجيل فوري
// تسجيل غير متزامن له القيمة: 777
```

### Thenables

قام نظام JavaScript البيئي بتنفيذ العديد من تطبيقات Promise قبل أن تصبح جزءاً من اللغة. على الرغم من تمثيلها بشكل مختلف داخلياً، في الحد الأدنى، تنفذ جميع الكائنات الشبيهة بـ Promise واجهة _Thenable_. يقوم thenable بتنفيذ طريقة [`.then()`](/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/then)، والتي يتم استدعاؤها مع دالتي callback: واحدة عندما يتم الوفاء بالـ promise، وواحدة عندما يتم رفضه. الـ Promises هي أيضاً thenables.

للتوافق مع تطبيقات Promise الموجودة، تسمح اللغة باستخدام thenables بدلاً من promises. على سبيل المثال، [`Promise.resolve`](/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/resolve) لن يقوم فقط بحل promises، ولكن أيضاً بتتبع thenables.

```js
const aThenable = {
  then(onFulfilled, onRejected) {
    onFulfilled({
      // يتم الوفاء بالـ thenable بـ thenable آخر
      then(onFulfilled, onRejected) {
        onFulfilled(42);
      },
    });
  },
};

Promise.resolve(aThenable); // promise تم الوفاء به بـ 42
```

### تزامن Promise

يقدم صنف `Promise` أربع طرق ثابتة لتسهيل [التزامن](https://en.wikipedia.org/wiki/Concurrent_computing) للمهام غير المتزامنة:

- {{jsxref("Promise.all()")}}
  - : يتم الوفاء به عندما يتم الوفاء بـ **جميع** الـ promises؛ يتم رفضه عندما يتم رفض **أي** من الـ promises.
- {{jsxref("Promise.allSettled()")}}
  - : يتم الوفاء به عندما تتم تسوية **جميع** الـ promises.
- {{jsxref("Promise.any()")}}
  - : يتم الوفاء به عندما يتم الوفاء بـ **أي** من الـ promises؛ يتم رفضه عندما يتم رفض **جميع** الـ promises.
- {{jsxref("Promise.race()")}}
  - : يتم تسويته عندما تتم تسوية **أي** من الـ promises. بمعنى آخر، يتم الوفاء به عندما يتم الوفاء بأي من الـ promises؛ يتم رفضه عندما يتم رفض أي من الـ promises.

تأخذ جميع هذه الطرق [iterable](/en-US/docs/Web/JavaScript/Reference/Iteration_protocols#the_iterable_protocol) من promises ([thenables](#thenables)، ليكون دقيقاً) وتُرجع promise جديداً. جميعها تدعم الوراثة الفرعية، مما يعني أنه يمكن استدعاؤها على الأصناف الفرعية لـ `Promise`، وستكون النتيجة promise من نوع الصنف الفرعي. للقيام بذلك، يجب أن ينفذ منشئ الصنف الفرعي نفس التوقيع مثل منشئ [`Promise()`](/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/Promise) - قبول دالة `executor` واحدة يمكن استدعاؤها مع callbacks `resolve` و `reject` كمعاملات. يجب أن يكون للصنف الفرعي أيضاً طريقة ثابتة `resolve` يمكن استدعاؤها مثل {{jsxref("Promise.resolve()")}} لحل القيم إلى promises.

لاحظ أن JavaScript [أحادي الخيط](/en-US/docs/Glossary/Thread) بطبيعته، لذلك في لحظة معينة، ستكون هناك مهمة واحدة فقط قيد التنفيذ، على الرغم من أن التحكم يمكن أن ينتقل بين promises مختلفة، مما يجعل تنفيذ الـ promises يبدو متزامناً. يمكن تحقيق [التنفيذ المتوازي](https://en.wikipedia.org/wiki/Parallel_computing) في JavaScript فقط من خلال [خيوط العمال](/en-US/docs/Web/API/Web_Workers_API).

## المنشئ

- {{jsxref("Promise/Promise", "Promise()")}}
  - : ينشئ كائن `Promise` جديد. يُستخدم المنشئ بشكل أساسي لتغليف الدوال التي لا تدعم promises بالفعل.

## الخصائص الثابتة

- [`Promise[Symbol.species]`](/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/Symbol.species)
  - : يُرجع المنشئ المستخدم لبناء القيم المُرجعة من طرق promise.

## الطرق الثابتة

- {{jsxref("Promise.all()")}}
  - : يأخذ iterable من promises كمدخل ويُرجع `Promise` واحداً. هذا الـ promise المُرجع يتم الوفاء به عندما يتم الوفاء بجميع promises المدخلة (بما في ذلك عندما يتم تمرير iterable فارغ)، مع مصفوفة من قيم الوفاء. يتم رفضه عندما يتم رفض أي من promises المدخلة، مع سبب الرفض الأول.
- {{jsxref("Promise.allSettled()")}}
  - : يأخذ iterable من promises كمدخل ويُرجع `Promise` واحداً. هذا الـ promise المُرجع يتم الوفاء به عندما تتم تسوية جميع promises المدخلة (بما في ذلك عندما يتم تمرير iterable فارغ)، مع مصفوفة من الكائنات التي تصف نتيجة كل promise.
- {{jsxref("Promise.any()")}}
  - : يأخذ iterable من promises كمدخل ويُرجع `Promise` واحداً. هذا الـ promise المُرجع يتم الوفاء به عندما يتم الوفاء بأي من promises المدخلة، مع قيمة الوفاء الأولى. يتم رفضه عندما يتم رفض جميع promises المدخلة (بما في ذلك عندما يتم تمرير iterable فارغ)، مع {{jsxref("AggregateError")}} يحتوي على مصفوفة من أسباب الرفض.
- {{jsxref("Promise.race()")}}
  - : يأخذ iterable من promises كمدخل ويُرجع `Promise` واحداً. هذا الـ promise المُرجع يتم تسويته مع الحالة النهائية لأول promise يتم تسويته.
- {{jsxref("Promise.reject()")}}
  - : يُرجع كائن `Promise` جديد تم رفضه مع السبب المعطى.
- {{jsxref("Promise.resolve()")}}
  - : يُرجع كائن `Promise` تم حله مع القيمة المعطاة. إذا كانت القيمة thenable (أي لديها طريقة `then`)، سيتبع الـ promise المُرجع ذلك الـ thenable، متبنياً حالته النهائية؛ وإلا، سيتم الوفاء بالـ promise المُرجع مع القيمة.
- {{jsxref("Promise.try()")}}
  - : يأخذ callback من أي نوع (يُرجع أو يرمي، بشكل متزامن أو غير متزامن) ويغلف نتيجته في `Promise`.
- {{jsxref("Promise.withResolvers()")}}
  - : يُرجع كائناً يحتوي على كائن `Promise` جديد ودالتين لحل أو رفض هذا الـ promise، مقابلتين للمعاملين الممررين إلى منفذ {{jsxref("Promise/Promise", "Promise()")}}.

## خصائص النسخة

هذه الخصائص معرفة على `Promise.prototype` ومشتركة بين جميع نسخ `Promise`.

- {{jsxref("Object/constructor", "Promise.prototype.constructor")}}
  - : دالة المنشئ التي أنشأت كائن النسخة. بالنسبة لنسخ `Promise`، القيمة الأولية هي منشئ {{jsxref("Promise/Promise", "Promise")}}.
- `Promise.prototype[Symbol.toStringTag]`
  - : القيمة الأولية لخاصية [`[Symbol.toStringTag]`](/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol/toStringTag) هي السلسلة `"Promise"`. تُستخدم هذه الخاصية في {{jsxref("Object.prototype.toString()")}}.

## طرق النسخة

- {{jsxref("Promise.prototype.catch()")}}
  - : يضيف معالج رفض إلى الـ promise، ويُرجع promise جديداً يتم حله إلى القيمة المُرجعة من المعالج إذا تم استدعاؤه، أو إلى قيمة الوفاء الأصلية إذا تم الوفاء بالـ promise بدلاً من ذلك.
- {{jsxref("Promise.prototype.finally()")}}
  - : يضيف معالجاً إلى الـ promise، ويُرجع promise جديداً يتم حله عندما يتم حل الـ promise الأصلي. يتم استدعاء المعالج عندما يتم تسوية الـ promise، سواء تم الوفاء به أو رفضه.
- {{jsxref("Promise.prototype.then()")}}
  - : يضيف معالجات الوفاء والرفض إلى الـ promise، ويُرجع promise جديداً يتم حله إلى القيمة المُرجعة من المعالج المستدعى، أو إلى قيمته المُسوَّاة الأصلية إذا لم تتم معالجة الـ promise (أي إذا لم يكن المعالج المعني `onFulfilled` أو `onRejected` دالة).

## أمثلة

### مثال أساسي

في هذا المثال، نستخدم `setTimeout(...)` لمحاكاة الكود غير المتزامن.
في الواقع، من المحتمل أن تستخدم شيئاً مثل XHR أو واجهة برمجة تطبيقات HTML.

```js
const myFirstPromise = new Promise((resolve, reject) => {
  // نستدعي resolve(...) عندما ينجح ما كنا نقوم به بشكل غير متزامن،
  // ونستدعي reject(...) عندما يفشل.
  setTimeout(() => {
    resolve("نجاح!"); // رائع! كل شيء سار بشكل جيد!
  }, 250);
});

myFirstPromise.then((successMessage) => {
  // successMessage هو أياً كان ما مررناه في دالة resolve(...) أعلاه.
  // ليس من الضروري أن تكون سلسلة نصية، ولكن إذا كانت مجرد رسالة نجاح، فمن المحتمل أن تكون كذلك.
  console.log(`رائع! ${successMessage}`);
});
```

### مثال مع مواقف متنوعة

يوضح هذا المثال تقنيات متنوعة لاستخدام إمكانيات Promise ومواقف متنوعة يمكن أن تحدث. لفهم هذا، ابدأ بالتمرير إلى أسفل كتلة الكود، وافحص سلسلة الـ promise. عند توفير promise أولي، يمكن أن تتبعه سلسلة من الـ promises. تتكون السلسلة من استدعاءات `.then()`، وعادةً (ولكن ليس بالضرورة) لديها `.catch()` واحد في النهاية، يتبعه اختيارياً `.finally()`. في هذا المثال، يتم بدء سلسلة الـ promise بواسطة بناء `new Promise()` مخصص؛ ولكن في الممارسة الفعلية، عادةً ما تبدأ سلاسل الـ promise بدالة واجهة برمجة تطبيقات (مكتوبة بواسطة شخص آخر) تُرجع promise.

توضح الدالة `tetheredGet Number()` أن منشئ promise سيستخدم `reject()` أثناء إعداد استدعاء غير متزامن، أو داخل الـ callback، أو كليهما. توضح الدالة `promiseGetWord()` كيف يمكن لدالة واجهة برمجة تطبيقات أن تنشئ وتُرجع promise بطريقة مستقلة.

لاحظ أن الدالة `troubleWithGetNumber()` تنتهي بـ `throw`. هذا مفروض لأن سلسلة promise تمر عبر جميع وعود `.then()`، حتى بعد حدوث خطأ، وبدون `throw`، سيبدو الخطأ "مُصلحاً". هذا أمر مزعج، ولهذا السبب، من الشائع حذف `onRejected` في سلسلة وعود `.then()`، والاكتفاء بـ `onRejected` واحد في `catch()` النهائي.

يمكن تشغيل هذا الكود تحت NodeJS. يتحسن الفهم برؤية الأخطاء تحدث بالفعل. لفرض المزيد من الأخطاء، قم بتغيير قيم `threshold`.

```js
// لتجربة معالجة الأخطاء، قيم "threshold" تسبب أخطاء عشوائية
const THRESHOLD_A = 8; // يمكن استخدام الصفر 0 لضمان حدوث خطأ

function tetheredGetNumber(resolve, reject) {
  setTimeout(() => {
    const randomInt = Date.now();
    const value = randomInt % 10;
    if (value < THRESHOLD_A) {
      resolve(value);
    } else {
      reject(`كبير جداً: ${value}`);
    }
  }, 500);
}

function determineParity(value) {
  const isOdd = value % 2 === 1;
  return { value, isOdd };
}

function troubleWithGetNumber(reason) {
  const err = new Error("مشكلة في الحصول على الرقم", { cause: reason });
  console.error(err);
  throw err;
}

function promiseGetWord(parityInfo) {
  return new Promise((resolve, reject) => {
    const { value, isOdd } = parityInfo;
    if (value >= THRESHOLD_A - 1) {
      reject(`ما زال كبيراً جداً: ${value}`);
    } else {
      parityInfo.wordEvenOdd = isOdd ? "فردي" : "زوجي";
      resolve(parityInfo);
    }
  });
}

new Promise(tetheredGetNumber)
  .then(determineParity, troubleWithGetNumber)
  .then(promiseGetWord)
  .then((info) => {
    console.log(`حصلنا على: ${info.value}، ${info.wordEvenOdd}`);
    return info;
  })
  .catch((reason) => {
    if (reason.cause) {
      console.error("تمت معالجة خطأ سابقاً");
    } else {
      console.error(`مشكلة مع promiseGetWord(): ${reason}`);
    }
  })
  .finally((info) => console.log("تم الانتهاء"));
```

### مثال متقدم

يوضح هذا المثال الصغير آلية `Promise`. يتم استدعاء طريقة `testPromise()` في كل مرة يتم فيها النقر على {{HTMLElement("button")}}. تنشئ promise سيتم الوفاء به، باستخدام {{domxref("Window.setTimeout", "setTimeout()")}}، بعدد الـ promise (رقم يبدأ من 1) كل 1-3 ثوانٍ، بشكل عشوائي. يُستخدم منشئ `Promise()` لإنشاء الـ promise.

يتم تسجيل الوفاء بالـ promise، عبر callback الوفاء المعين باستخدام {{jsxref("Promise/then", "p1.then()")}}. تُظهر بعض السجلات كيف يتم فصل الجزء المتزامن من الطريقة عن الإكمال غير المتزامن للـ promise.

من خلال النقر على الزر عدة مرات في فترة زمنية قصيرة، سترى حتى الـ promises المختلفة يتم الوفاء بها واحداً تلو الآخر.

#### HTML

```html
<button id="make-promise">قم بإنشاء promise!</button>
<div id="log"></div>
```

#### JavaScript

```js
"use strict";

let promiseCount = 0;

function testPromise() {
  const thisPromiseCount = ++promiseCount;
  const log = document.getElementById("log");
  // البداية
  log.insertAdjacentHTML("beforeend", `${thisPromiseCount}) بدأ<br>`);
  // نقوم بإنشاء promise جديد: نعد بعدد رقمي لهذا الـ promise،
  // بدءاً من 1 (بعد الانتظار 3 ثوانٍ)
  const p1 = new Promise((resolve, reject) => {
    // يتم استدعاء دالة المنفذ مع القدرة
    // على حل أو رفض الـ promise
    log.insertAdjacentHTML(
      "beforeend",
      `${thisPromiseCount}) منشئ Promise<br>`,
    );
    // هذا مجرد مثال لإنشاء عدم التزامن
    setTimeout(
      () => {
        // نقوم بالوفاء بالـ promise
        resolve(thisPromiseCount);
      },
      Math.random() * 2000 + 1000,
    );
  });

  // نحدد ما يجب فعله عندما يتم حل الـ promise باستدعاء then()،
  // وما يجب فعله عندما يتم رفض الـ promise باستدعاء catch()
  p1.then((val) => {
    // تسجيل قيمة الوفاء
    log.insertAdjacentHTML("beforeend", `${val}) تم الوفاء بالـ Promise<br>`);
  }).catch((reason) => {
    // تسجيل سبب الرفض
    console.log(`معالجة الـ promise المرفوض (${reason}) هنا.`);
  });
  // النهاية
  log.insertAdjacentHTML("beforeend", `${thisPromiseCount}) تم إنشاء Promise<br>`);
}

const btn = document.getElementById("make-promise");
btn.addEventListener("click", testPromise);
```

#### النتيجة

{{EmbedLiveSample("مثال_متقدم", "500", "200")}}

### تحميل صورة باستخدام XHR

مثال آخر يستخدم `Promise` و {{domxref("XMLHttpRequest")}} لتحميل صورة موضح أدناه.
تم التعليق على كل خطوة ويسمح لك بمتابعة بنية Promise و XHR عن كثب.

```html hidden live-sample___promises
<h1>مثال Promise</h1>
```

```js live-sample___promises
function imgLoad(url) {
  // إنشاء promise جديد باستخدام منشئ Promise()؛
  // لديه كمعامل دالة مع معاملين، resolve و reject
  return new Promise((resolve, reject) => {
    // XHR لتحميل صورة
    const request = new XMLHttpRequest();
    request.open("GET", url);
    request.responseType = "blob";
    // عندما يتم تحميل الطلب، تحقق مما إذا كان ناجحاً
    request.onload = () => {
      if (request.status === 200) {
        // إذا نجح، قم بحل الـ promise عن طريق تمرير استجابة الطلب
        resolve(request.response);
      } else {
        // إذا فشل، ارفض الـ promise مع رسالة خطأ
        reject(
          Error(
            `لم يتم تحميل الصورة بنجاح؛ رمز الخطأ: ${request.statusText}`,
          ),
        );
      }
    };
    // معالجة أخطاء الشبكة
    request.onerror = () => reject(new Error("كان هناك خطأ في الشبكة."));
    // إرسال الطلب
    request.send();
  });
}

// الحصول على مرجع لعنصر body، وإنشاء كائن صورة جديد
const body = document.querySelector("body");
const myImage = new Image();
const imgUrl =
  "https://mdn.github.io/shared-assets/images/examples/round-balloon.png";

// استدعاء الدالة مع عنوان URL الذي نريد تحميله، ثم ربط
// طريقة then() للـ promise مع دالتي callback
imgLoad(imgUrl).then(
  (response) => {
    // الأولى تعمل عندما يتم حل الـ promise، مع request.response
    // المحدد داخل طريقة resolve().
    const imageURL = URL.createObjectURL(response);
    myImage.src = imageURL;
    body.appendChild(myImage);
  },
  (error) => {
    // الثانية تعمل عندما يتم رفض الـ promise،
    // وتسجل الخطأ المحدد مع طريقة reject().
    console.log(error);
  },
);
```

{{embedlivesample("promises", "", "240px")}}

### تتبع كائن الإعدادات الحالي

كائن الإعدادات هو [بيئة](https://html.spec.whatwg.org/multipage/webappapis.html#environment-settings-object) توفر معلومات إضافية عندما يتم تشغيل كود JavaScript. يتضمن ذلك المجال وخريطة الوحدات، بالإضافة إلى معلومات خاصة بـ HTML مثل الأصل. يتم تتبع كائن الإعدادات الحالي لضمان أن المتصفح يعرف أيها يجب استخدامه لقطعة معينة من كود المستخدم.

لتصور هذا بشكل أفضل، يمكننا إلقاء نظرة أقرب على كيف يمكن أن يكون المجال مشكلة. يمكن اعتبار **المجال** تقريباً كالكائن العالمي. ما يميز المجالات هو أنها تحتفظ بجميع المعلومات الضرورية لتشغيل كود JavaScript. يتضمن ذلك كائنات مثل [`Array`](/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array) و [`Error`](/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error). لكل كائن إعدادات "نسخته" الخاصة من هذه وهي غير مشتركة. يمكن أن يسبب ذلك بعض السلوك غير المتوقع فيما يتعلق بالـ promises. للتغلب على هذا، نتتبع شيئاً يسمى **كائن الإعدادات الحالي**. يمثل هذا معلومات خاصة بسياق كود المستخدم المسؤول عن استدعاء دالة معينة.

لتوضيح هذا أكثر يمكننا إلقاء نظرة على كيف يتواصل [`<iframe>`](/en-US/docs/Web/HTML/Element/iframe) مضمن في مستند مع مضيفه. نظراً لأن جميع واجهات برمجة التطبيقات الويب تدرك كائن الإعدادات الحالي، سيعمل ما يلي في جميع المتصفحات:

```html
<!doctype html> <iframe></iframe>
<!-- لدينا مجال هنا -->
<script>
  // لدينا مجال هنا أيضاً
  const bound = frames[0].postMessage.bind(frames[0], "بعض البيانات", "*");
  // bound هي دالة مدمجة - لا يوجد كود مستخدم
  // على المكدس، فأي مجال نستخدم؟
  setTimeout(bound);
  // هذا لا يزال يعمل، لأننا نستخدم المجال الأحدث
  // (الحالي) على المكدس
</script>
```

ينطبق نفس المفهوم على الـ promises. إذا قمنا بتعديل المثال أعلاه قليلاً، نحصل على هذا:

```html
<!doctype html> <iframe></iframe>
<!-- لدينا مجال هنا -->
<script>
  // لدينا مجال هنا أيضاً
  const bound = frames[0].postMessage.bind(frames[0], "بعض البيانات", "*");
  // bound هي دالة مدمجة - لا يوجد كود مستخدم
  // على المكدس - أي مجال نستخدم؟
  Promise.resolve(undefined).then(bound);
  // هذا لا يزال يعمل، لأننا نستخدم المجال الأحدث
  // (الحالي) على المكدس
</script>
```

إذا غيرنا هذا بحيث يستمع الـ `<iframe>` في المستند إلى الرسائل المرسلة، يمكننا ملاحظة تأثير كائن الإعدادات الحالي:

```html
<!-- y.html -->
<!doctype html>
<iframe src="x.html"></iframe>
<script>
  const bound = frames[0].postMessage.bind(frames[0], "بعض البيانات", "*");
  Promise.resolve(undefined).then(bound);
</script>
```

```html
<!-- x.html -->
<!doctype html>
<script>
  window.addEventListener(
    "message",
    (event) => {
      document.querySelector("#text").textContent = "مرحباً";
      // سيعمل هذا الكود فقط في المتصفحات التي تتتبع كائن الإعدادات الحالي
      console.log(event);
    },
    false,
  );
</script>
```

في المثال أعلاه، سيتم تحديث النص الداخلي للـ `<iframe>` فقط إذا تم تتبع كائن الإعدادات الحالي. هذا لأنه بدون تتبع الحالي، قد ننتهي باستخدام البيئة الخاطئة لإرسال الرسالة.

> [!NOTE]
> حالياً، تتبع المجال الحالي مُنفذ بالكامل في Firefox، ولديه تنفيذات جزئية في Chrome و Safari.

## المواصفات

{{Specifications}}

## توافق المتصفحات

{{Compat}}

## انظر أيضاً

- [Polyfill لـ `Promise` في `core-js`](https://github.com/zloirock/core-js#ecmascript-promise)
- دليل [استخدام promises](/en-US/docs/Web/JavaScript/Guide/Using_promises)
- [مواصفات Promises/A+](https://promisesaplus.com/)
- [مقدمة إلى JavaScript Promises](https://web.dev/articles/promises) على web.dev (2013)
- [Callbacks, Promises, والـ Coroutines: أنماط البرمجة غير المتزامنة في JavaScript](https://www.slideshare.net/slideshow/callbacks-promises-and-coroutines-oh-my-the-evolution-of-asynchronicity-in-javascript/9953720) عرض شرائح بواسطة Domenic Denicola (2011)