---
title: extends
slug: Web/JavaScript/Reference/Classes/extends  
page-type: javascript-language-feature
browser-compat: javascript.classes.extends
---

# الكلمة المفتاحية extends

تُستخدم الكلمة المفتاحية **`extends`** في تصريحات الـ class أو تعبيرات الـ class لإنشاء class فرعي (child) يرث من class آخر (parent).

## بناء الجملة

```js-nolint
class ChildClass extends ParentClass { /* … */ }
```

- `ParentClass`
  - : تعبير يُقيَّم إلى دالة constructor (بما في ذلك class) أو `null`.

## الوصف

يمكن استخدام الكلمة المفتاحية `extends` للوراثة من classes مخصصة وكذلك من الكائنات المدمجة في JavaScript.

أي constructor يمكن استدعاؤه باستخدام `new` ولديه خاصية `prototype` يمكن أن يكون مرشحاً ليكون الـ class الأب. يجب توفر كلا الشرطين - على سبيل المثال، الدوال المقيدة (bound functions) و `Proxy` يمكن إنشاؤها، لكن ليس لديها خاصية `prototype`، لذا لا يمكن الوراثة منها.

```js
function OldStyleClass() {
  this.someProperty = 1;
}
OldStyleClass.prototype.someMethod = function () {};

class ChildClass extends OldStyleClass {}

class ModernClass {
  someProperty = 1;
  someMethod() {}
}

class AnotherChildClass extends ModernClass {}
```

يجب أن تكون خاصية `prototype` للـ `ParentClass` عبارة عن `Object` أو `null`، لكنك نادراً ما ستقلق بشأن هذا عملياً، لأن الـ `prototype` غير الكائني لا يتصرف كما ينبغي على أي حال. (يتم تجاهله بواسطة عامل التشغيل `new`).

```js
function ParentClass() {}
ParentClass.prototype = 3;

class ChildClass extends ParentClass {}
// Uncaught TypeError: Class extends value does not have valid prototype property 3

console.log(Object.getPrototypeOf(new ParentClass()));
// [Object: null prototype] {}
// ليس رقماً في الواقع!
```

تقوم `extends` بتعيين الـ prototype لكل من `ChildClass` و `ChildClass.prototype`.

|                                   | Prototype لـ `ChildClass` | Prototype لـ `ChildClass.prototype` |
| --------------------------------- | ------------------------- | ----------------------------------- |
| عدم وجود `extends`                | `Function.prototype`      | `Object.prototype`                  |
| [`extends null`](#extending_null)  | `Function.prototype`      | `null`                             |
| `extends ParentClass`             | `ParentClass`             | `ParentClass.prototype`             |

```js
class ParentClass {}
class ChildClass extends ParentClass {}

// يسمح بوراثة الخصائص الثابتة (static)
Object.getPrototypeOf(ChildClass) === ParentClass;
// يسمح بوراثة خصائص النسخة (instance)
Object.getPrototypeOf(ChildClass.prototype) === ParentClass.prototype;
```

الجانب الأيمن من `extends` لا يجب أن يكون معرفاً. يمكنك استخدام أي تعبير يُقيَّم إلى constructor. هذا مفيد غالباً لإنشاء mixins. قيمة `this` في تعبير `extends` هي `this` المحيطة بتعريف الـ class، والإشارة إلى اسم الـ class تؤدي إلى `ReferenceError` لأن الـ class لم يتم تهيئته بعد. `await` و `yield` يعملان كما هو متوقع في هذا التعبير.

```js
class SomeClass extends class {
  constructor() {
    console.log("Base class");
  }
} {
  constructor() {
    super();
    console.log("Derived class");
  }
}

new SomeClass();
// Base class
// Derived class
```

بينما يمكن للـ class الأب أن يُرجع أي شيء من الـ constructor الخاص به، يجب على الـ class الفرعي أن يُرجع كائناً أو `undefined`، وإلا سيتم إلقاء خطأ `TypeError`.

```js
class ParentClass {
  constructor() {
    return 1;
  }
}

console.log(new ParentClass()); // ParentClass {}
// يتم تجاهل القيمة المُرجعة لأنها ليست كائناً
// وهذا يتوافق مع سلوك دوال الـ constructor

class ChildClass extends ParentClass {
  constructor() {
    super();
    return 1;
  }
}

console.log(new ChildClass()); // TypeError: يمكن للـ constructors الفرعية أن تُرجع فقط كائناً أو undefined
```

إذا أرجع constructor الـ class الأب كائناً، سيتم استخدام هذا الكائن كقيمة `this` للـ class الفرعي عند تهيئة حقول الـ class لاحقاً. هذه الحيلة تُسمى "return overriding"، والتي تسمح بتعريف حقول الـ class الفرعي (بما في ذلك الحقول الخاصة) على كائنات غير مرتبطة.

### الوراثة من الكائنات المدمجة

> [!تحذير]
> تعتبر اللجنة القياسية الآن أن آلية الوراثة من الكائنات المدمجة في الإصدارات السابقة من المواصفات معقدة بشكل مفرط وتسبب تأثيرات ملحوظة على الأداء والأمان. الطرق المدمجة الجديدة تأخذ في الاعتبار بشكل أقل الـ classes الفرعية، ومنفذو المحركات يدرسون إمكانية إزالة بعض آليات الوراثة. يُفضل استخدام التركيب (composition) بدلاً من الوراثة عند تحسين الكائنات المدمجة.

هناك بعض الأمور التي قد تتوقعها عند الوراثة من class:

- عند استدعاء طريقة إنشاء ثابتة (مثل `Promise.resolve()` أو `Array.from()`) على class فرعي، تكون النسخة المُرجعة دائماً نسخة من الـ class الفرعي.
- عند استدعاء طريقة نسخة تُرجع نسخة جديدة (مثل `Promise.prototype.then()` أو `Array.prototype.map()`) على class فرعي، تكون النسخة المُرجعة دائماً نسخة من الـ class الفرعي.
- تحاول طرق النسخة تفويض العمليات إلى مجموعة صغيرة من الطرق الأساسية حيثما أمكن. على سبيل المثال، بالنسبة لـ class فرعي من `Promise`، تعديل `then()` يؤدي تلقائياً إلى تغيير سلوك `catch()`؛ أو بالنسبة لـ class فرعي من `Map`، تعديل `set()` يؤدي تلقائياً إلى تغيير سلوك constructor الـ `Map()`.

ومع ذلك، تتطلب التوقعات المذكورة أعلاه جهوداً كبيرة للتنفيذ بشكل صحيح.

- الأول يتطلب أن تقرأ الطريقة الثابتة قيمة `this` للحصول على الـ constructor لإنشاء النسخة المُرجعة. هذا يعني أن `[p1, p2, p3].map(Promise.resolve)` يُلقي خطأً لأن `this` داخل `Promise.resolve` هو `undefined`. طريقة لإصلاح هذا هي الرجوع إلى الـ class الأساسي إذا لم يكن `this` constructor، كما يفعل `Array.from()`، لكن هذا يعني أن الـ class الأساسي لا يزال يُعامل معاملة خاصة.
- الثاني يتطلب أن تقرأ طريقة النسخة `this.constructor` للحصول على دالة الـ constructor. ومع ذلك، قد يؤدي `new this.constructor()` إلى كسر الكود القديم، لأن خاصية `constructor` قابلة للكتابة والتكوين ولا تتمتع بأي حماية. لذلك، تستخدم العديد من طرق النسخ المدمجة خاصية `[Symbol.species]` للـ constructor بدلاً من ذلك (والتي تُرجع افتراضياً `this`، أي الـ constructor نفسه). ومع ذلك، يسمح `[Symbol.species]` بتشغيل كود عشوائي وإنشاء نسخ من أي نوع، مما يشكل مخاوف أمنية ويعقد دلالات الوراثة بشكل كبير.
- الثالث يؤدي إلى استدعاءات مرئية للكود المخصص، مما يجعل العديد من التحسينات أصعب في التنفيذ. على سبيل المثال، إذا تم استدعاء constructor الـ `Map()` مع iterable من _x_ عناصر، فيجب عليه استدعاء طريقة `set()` بشكل مرئي _x_ مرات، بدلاً من مجرد نسخ العناصر إلى التخزين الداخلي.

هذه المشاكل ليست فريدة من نوعها للـ classes المدمجة. بالنسبة لـ classes الخاصة بك، ستحتاج على الأرجح إلى اتخاذ نفس القرارات. ومع ذلك، بالنسبة للـ classes المدمجة، تعتبر قابلية التحسين والأمان مصدر قلق أكبر بكثير. الطرق المدمجة الجديدة تقوم دائماً بإنشاء الـ class الأساسي واستدعاء أقل عدد ممكن من الطرق المخصصة. إذا كنت تريد الوراثة من الكائنات المدمجة مع تحقيق التوقعات المذكورة أعلاه، فستحتاج إلى تجاوز جميع الطرق التي لديها السلوك الافتراضي المدمج فيها. أي إضافة لطرق جديدة على الـ class الأساسي قد تكسر أيضاً دلالات الـ class الفرعي الخاص بك لأنها تُورث افتراضياً. لذلك، طريقة أفضل لتوسيع الكائنات المدمجة هي استخدام [_التركيب_](#avoiding_inheritance).

### الوراثة من null

تم تصميم `extends null` للسماح بإنشاء سهل للكائنات التي لا ترث من `Object.prototype`. ومع ذلك، بسبب القرارات غير المستقرة حول ما إذا كان يجب استدعاء `super()` داخل الـ constructor، لا يمكن إنشاء مثل هذا الـ class عملياً باستخدام أي تنفيذ للـ constructor لا يُرجع كائناً. [تعمل لجنة TC39 على إعادة تمكين هذه الميزة](https://github.com/tc39/ecma262/pull/1321).

```js
new (class extends null {})();
// TypeError: Super constructor null of anonymous class is not a constructor

new (class extends null {
  constructor() {}
})();
// ReferenceError: Must call super constructor in derived class before accessing 'this' or returning from derived constructor

new (class extends null {
  constructor() {
    super();
  }
})();
// TypeError: Super constructor null of anonymous class is not a constructor
```

بدلاً من ذلك، تحتاج إلى إرجاع نسخة بشكل صريح من الـ constructor.

```js
class NullClass extends null {
  constructor() {
    // استخدام new.target يسمح للـ classes الفرعية
    // بأن يكون لديها سلسلة prototype صحيحة
    return Object.create(new.target.prototype);
  }
}

const proto = Object.getPrototypeOf;
console.log(proto(proto(new NullClass()))); // null
```

## أمثلة

### استخدام extends

المثال الأول ينشئ class يُسمى `Square` من class يُسمى `Polygon`. هذا المثال مستخرج من [هذا العرض التوضيحي المباشر](https://googlechrome.github.io/samples/classes-es6/index.html) [(المصدر)](https://github.com/GoogleChrome/samples/blob/gh-pages/classes-es6/index.html).

```js
class Square extends Polygon {
  constructor(length) {
    // هنا، يتم استدعاء constructor الـ class الأب مع الأطوال
    // المقدمة لعرض وارتفاع المضلع
    super(length, length);
    // ملاحظة: في الـ classes الفرعية، يجب استدعاء super() قبل
    // أن تتمكن من استخدام 'this'. حذف هذا سيؤدي إلى خطأ مرجعي.
    this.name = "Square";
  }

  get area() {
    return this.height * this.width;
  }
}
```

### الوراثة من الكائنات العادية

لا يمكن للـ classes الوراثة من الكائنات العادية (غير القابلة للإنشاء). إذا كنت تريد الوراثة من كائن عادي بجعل جميع خصائص هذا الكائن متاحة في النسخ الموروثة، يمكنك بدلاً من ذلك استخدام `Object.setPrototypeOf()`:

```js
const Animal = {
  speak() {
    console.log(`${this.name} يصدر صوتاً.`);
  },
};

class Dog {
  constructor(name) {
    this.name = name;
  }
}

Object.setPrototypeOf(Dog.prototype, Animal);

const d = new Dog("ميتزي");
d.speak(); // ميتزي يصدر صوتاً.
```

### الوراثة من الكائنات المدمجة

هذا المثال يرث من الكائن المدمج `Date`. هذا المثال مستخرج من [هذا العرض التوضيحي المباشر](https://googlechrome.github.io/samples/classes-es6/index.html) [(المصدر)](https://github.com/GoogleChrome/samples/blob/gh-pages/classes-es6/index.html).

```js
class MyDate extends Date {
  getFormattedDate() {
    const months = [
      "يناير", "فبراير", "مارس", "أبريل", "مايو", "يونيو",
      "يوليو", "أغسطس", "سبتمبر", "أكتوبر", "نوفمبر", "ديسمبر",
    ];
    return `${this.getDate()}-${months[this.getMonth()]}-${this.getFullYear()}`;
  }
}
```

### الوراثة من Object

جميع كائنات JavaScript ترث من `Object.prototype` بشكل افتراضي، لذا فإن كتابة `extends Object` قد تبدو زائدة عن الحاجة للوهلة الأولى. الفرق الوحيد عن عدم كتابة `extends` على الإطلاق هو أن الـ constructor نفسه يرث الطرق الثابتة من `Object`، مثل `Object.keys()`. ومع ذلك، لأنه لا توجد طريقة ثابتة من `Object` تستخدم قيمة `this`، فلا تزال لا توجد قيمة في وراثة هذه الطرق الثابتة.

يتعامل constructor الـ `Object()` بشكل خاص مع سيناريو الوراثة. إذا تم استدعاؤه ضمنياً عبر `super()`، فإنه دائماً يقوم بتهيئة كائن جديد مع `new.target.prototype` كـ prototype له. يتم تجاهل أي قيمة تُمرر إلى `super()`.

```js
class C extends Object {
  constructor(v) {
    super(v);
  }
}

console.log(new C(1) instanceof Number); // false
console.log(C.keys({ a: 1, b: 2 })); // [ 'a', 'b' ]
```

قارن هذا السلوك مع wrapper مخصص لا يتعامل بشكل خاص مع الوراثة:

```js
function MyObject(v) {
  return new Object(v);
}
class D extends MyObject {
  constructor(v) {
    super(v);
  }
}
console.log(new D(1) instanceof Number); // true
```

### Species

قد ترغب في إرجاع كائنات `Array` في class المصفوفة المشتق الخاص بك `MyArray`. نمط الـ species يتيح لك تجاوز الـ constructors الافتراضية.

على سبيل المثال، عند استخدام طرق مثل `Array.prototype.map()` التي تُرجع الـ constructor الافتراضي، تريد أن تُرجع هذه الطرق كائن `Array` أصلي، بدلاً من كائن `MyArray`. يتيح لك الرمز `Symbol.species` القيام بذلك:

```js
class MyArray extends Array {
  // تجاوز species إلى constructor المصفوفة الأصلي
  static get [Symbol.species]() {
    return Array;
  }
}

const a = new MyArray(1, 2, 3);
const mapped = a.map((x) => x * x);

console.log(mapped instanceof MyArray); // false
console.log(mapped instanceof Array); // true
```

يتم تنفيذ هذا السلوك بواسطة العديد من طرق النسخ المدمجة. للتحذيرات حول هذه الميزة، راجع مناقشة [الوراثة من الكائنات المدمجة](#subclassing_built-ins).

### Mix-ins

الـ classes الفرعية المجردة أو _mix-ins_ هي قوالب للـ classes. يمكن للـ class أن يكون له superclass واحد فقط، لذا فإن الوراثة المتعددة من classes الأدوات، على سبيل المثال، غير ممكنة. يجب توفير الوظائف بواسطة الـ superclass.

يمكن استخدام دالة مع superclass كمدخل وclass فرعي يرث من ذلك الـ superclass كمخرج لتنفيذ mix-ins:

```js
const calculatorMixin = (Base) =>
  class extends Base {
    calc() {}
  };

const randomizerMixin = (Base) =>
  class extends Base {
    randomize() {}
  };
```

يمكن كتابة class يستخدم هذه الـ mix-ins كما يلي:

```js
class Foo {}
class Bar extends calculatorMixin(randomizerMixin(Foo)) {}
```

### تجنب الوراثة

الوراثة هي علاقة اقتران قوية في البرمجة كائنية التوجه. تعني أن جميع سلوكيات الـ class الأساسي يتم وراثتها بواسطة الـ class الفرعي بشكل افتراضي، وهو ما قد لا يكون دائماً ما تريده. على سبيل المثال، انظر إلى تنفيذ `ReadOnlyMap`:

```js
class ReadOnlyMap extends Map {
  set() {
    throw new TypeError("يجب تعيين خريطة للقراءة فقط في وقت الإنشاء.");
  }
}
```

يتبين أن `ReadOnlyMap` غير قابل للإنشاء، لأن constructor الـ `Map()` يستدعي طريقة `set()` للنسخة.

```js
const m = new ReadOnlyMap([["a", 1]]); // TypeError: يجب تعيين خريطة للقراءة فقط في وقت الإنشاء.
```

قد نتجاوز هذا باستخدام علامة خاصة للإشارة إلى ما إذا كانت النسخة قيد الإنشاء. ومع ذلك، المشكلة الأكبر مع هذا التصميم هي أنه يكسر [مبدأ استبدال Liskov](https://en.wikipedia.org/wiki/Liskov_substitution_principle)، الذي ينص على أنه يجب أن يكون الـ class الفرعي قابلاً للاستبدال بـ superclass الخاص به. إذا كانت دالة تتوقع كائن `Map`، يجب أن تكون قادرة على استخدام كائن `ReadOnlyMap` أيضاً، وهو ما سينكسر هنا.

غالباً ما تؤدي الوراثة إلى [مشكلة الدائرة-البيضاوي](https://en.wikipedia.org/wiki/Circle%E2%80%93ellipse_problem)، لأن أياً من النوعين لا يتضمن بشكل مثالي سلوك الآخر، على الرغم من أنهما يشتركان في العديد من السمات المشتركة. بشكل عام، ما لم يكن هناك سبب وجيه لاستخدام الوراثة، من الأفضل استخدام التركيب بدلاً من ذلك. التركيب يعني أن الـ class لديه مرجع لكائن من class آخر، ويستخدم ذلك الكائن فقط كتفصيل تنفيذي.

```js
class ReadOnlyMap {
  #data;
  constructor(values) {
    this.#data = new Map(values);
  }
  get(key) {
    return this.#data.get(key);
  }
  has(key) {
    return this.#data.has(key);
  }
  get size() {
    return this.#data.size;
  }
  *keys() {
    yield* this.#data.keys();
  }
  *values() {
    yield* this.#data.values();
  }
  *entries() {
    yield* this.#data.entries();
  }
  *[Symbol.iterator]() {
    yield* this.#data[Symbol.iterator]();
  }
}
```

في هذه الحالة، class الـ `ReadOnlyMap` ليس class فرعياً من `Map`، لكنه لا يزال ينفذ معظم نفس الطرق. هذا يعني المزيد من تكرار الكود، لكنه يعني أيضاً أن class الـ `ReadOnlyMap` غير مرتبط بشدة بـ class الـ `Map`، ولا ينكسر بسهولة إذا تم تغيير class الـ `Map`، متجنباً [المشاكل الدلالية للوراثة من الكائنات المدمجة](#subclassing_built-ins). على سبيل المثال، إذا أضاف class الـ `Map` طريقة [`emplace()`](https://github.com/tc39/proposal-upsert) التي لا تستدعي `set()`، فإن ذلك سيؤدي إلى أن class الـ `ReadOnlyMap` لم يعد للقراءة فقط ما لم يتم تحديثه وفقاً لذلك لتجاوز `emplace()` أيضاً. علاوة على ذلك، كائنات `ReadOnlyMap` ليس لديها طريقة `set` على الإطلاق، وهذا أكثر دقة من إلقاء خطأ في وقت التشغيل.

## المواصفات

{{Specifications}}

## توافق المتصفحات

{{Compat}}

## انظر أيضاً

- [دليل استخدام classes](/en-US/docs/Web/JavaScript/Guide/Using_classes)
- [Classes](/en-US/docs/Web/JavaScript/Reference/Classes)
- {{jsxref("Classes/constructor", "constructor")}}
- {{jsxref("Statements/class", "class")}}
- {{jsxref("Operators/super", "super")}}