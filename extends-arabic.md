# extends
متوفر على نطاق واسع كخط أساسي

تُستخدم كلمة extends في تصريحات الـ class أو تعبيرات الـ class لإنشاء class يكون بمثابة class فرعي من class آخر.

جربها

```js
class DateFormatter extends Date {
  getFormattedDate() {
    const months = [
      'Jan',
      'Feb',
      'Mar',
      'Apr',
      'May',
      'Jun',
      'Jul',
      'Aug',
      'Sep',
      'Oct',
      'Nov',
      'Dec',
    ];
    return `${this.getDate()}-${months[this.getMonth()]}-${this.getFullYear()}`;
  }
}

console.log(new DateFormatter('August 19, 1975 23:15:30').getFormattedDate());
// النتيجة المتوقعة: "19-Aug-1975"
```

## بناء الجملة (Syntax)
```js
class ChildClass extends ParentClass { /* ... */ }
```

ParentClass
تعبير يتم تقييمه إلى دالة constructor (بما في ذلك class) أو null.

## الوصف
يمكن استخدام كلمة extends للوراثة من classes مخصصة وكذلك الكائنات المدمجة في اللغة.

أي constructor يمكن استدعاؤه باستخدام new ولديه خاصية prototype يمكن أن يكون مرشحاً ليكون الـ class الأب. يجب توفر كلا الشرطين - على سبيل المثال، الدوال المقيدة (bound functions) و Proxy يمكن إنشاؤها، لكن ليس لديها خاصية prototype، لذا لا يمكن الوراثة منها.

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

يجب أن تكون خاصية prototype للـ ParentClass عبارة عن Object أو null، لكنك نادراً ما ستقلق بشأن هذا في الممارسة العملية، لأن الـ prototype غير الكائني لا يتصرف كما ينبغي على أي حال. (يتم تجاهله من قبل المشغل new.)

```js
function ParentClass() {}
ParentClass.prototype = 3;

class ChildClass extends ParentClass {}
// خطأ: قيمة Class extends لا تحتوي على خاصية prototype صالحة 3

console.log(Object.getPrototypeOf(new ParentClass()));
// [Object: null prototype] {}
// ليس رقماً في الواقع!
```

extends تحدد الـ prototype لكل من ChildClass و ChildClass.prototype.

| عبارة extends | Prototype لـ ChildClass | Prototype لـ ChildClass.prototype |
|---------------|------------------------|----------------------------------|
| غير موجودة | Function.prototype | Object.prototype |
| extends null | Function.prototype | null |
| extends ParentClass | ParentClass | ParentClass.prototype |

```js
class ParentClass {}
class ChildClass extends ParentClass {}

// يسمح بوراثة الخصائص الثابتة
Object.getPrototypeOf(ChildClass) === ParentClass;
// يسمح بوراثة خصائص النسخة
Object.getPrototypeOf(ChildClass.prototype) === ParentClass.prototype;
```

الجانب الأيمن من extends لا يجب أن يكون معرفاً. يمكنك استخدام أي تعبير يتم تقييمه إلى constructor. هذا مفيد غالباً لإنشاء mixins. قيمة this في تعبير extends هي this المحيطة بتعريف الـ class، والإشارة إلى اسم الـ class تؤدي إلى ReferenceError لأن الـ class لم يتم تهيئته بعد. تعمل await و yield كما هو متوقع في هذا التعبير.

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

في حين أن الـ class الأساسي يمكن أن يعيد أي شيء من constructor الخاص به، يجب أن يعيد الـ class المشتق كائناً أو undefined، وإلا سيتم إلقاء TypeError.

```js
class ParentClass {
  constructor() {
    return 1;
  }
}

console.log(new ParentClass()); // ParentClass {}
// يتم تجاهل قيمة العودة لأنها ليست كائناً
// هذا متوافق مع دوال constructor

class ChildClass extends ParentClass {
  constructor() {
    super();
    return 1;
  }
}

console.log(new ChildClass()); // TypeError: يمكن للـ constructors المشتقة أن تعيد فقط كائناً أو undefined
```

إذا أعاد constructor الـ class الأب كائناً، سيتم استخدام هذا الكائن كقيمة this للـ class المشتق عند تهيئة حقول الـ class لاحقاً. هذه الحيلة تسمى "return overriding"، والتي تسمح بتعريف حقول الـ class المشتق (بما في ذلك الحقول الخاصة) على كائنات غير مرتبطة.

## الوراثة من الكائنات المدمجة
تحذير: تعتبر اللجنة القياسية الآن أن آلية الوراثة من الكائنات المدمجة في الإصدارات السابقة من المواصفات معقدة بشكل مفرط وتسبب تأثيرات ملحوظة على الأداء والأمان. الطرق المدمجة الجديدة تأخذ في الاعتبار بشكل أقل الـ classes الفرعية، ومنفذو المحركات يبحثون في إمكانية إزالة بعض آليات الوراثة. فكر في استخدام التركيب (composition) بدلاً من الوراثة عند تحسين الكائنات المدمجة.

إليك بعض الأمور التي قد تتوقعها عند الوراثة من class:

1. عند استدعاء طريقة إنشاء ثابتة (مثل Promise.resolve() أو Array.from()) على class فرعي، تكون النسخة المُرجعة دائماً نسخة من الـ class الفرعي.
2. عند استدعاء طريقة نسخة تُرجع نسخة جديدة (مثل Promise.prototype.then() أو Array.prototype.map()) على class فرعي، تكون النسخة المُرجعة دائماً نسخة من الـ class الفرعي.
3. تحاول طرق النسخة تفويض العمليات إلى مجموعة صغيرة من الطرق الأساسية حيثما أمكن ذلك.

ومع ذلك، تتطلب التوقعات المذكورة أعلاه جهوداً كبيرة للتنفيذ بشكل صحيح.

## الوراثة من null
تم تصميم extends null للسماح بإنشاء كائنات لا ترث من Object.prototype بسهولة. ومع ذلك، بسبب القرارات غير المستقرة حول ما إذا كان يجب استدعاء super() داخل الـ constructor، لا يمكن إنشاء مثل هذا الـ class عملياً باستخدام أي تنفيذ constructor لا يعيد كائناً. تعمل لجنة TC39 على إعادة تمكين هذه الميزة.

```js
new (class extends null {})();
// TypeError: Super constructor null of anonymous class is not a constructor

new (class extends null {
  constructor() {}
})();
// ReferenceError: يجب استدعاء super constructor في الـ class المشتق قبل الوصول إلى 'this' أو العودة من constructor المشتق

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
    // استخدام new.target يسمح للـ classes المشتقة
    // بأن يكون لديها سلسلة prototype صحيحة
    return Object.create(new.target.prototype);
  }
}

const proto = Object.getPrototypeOf;
console.log(proto(proto(new NullClass()))); // null
```

## أمثلة
### استخدام extends
المثال الأول ينشئ class يسمى Square من class يسمى Polygon.

```js
class Square extends Polygon {
  constructor(length) {
    // هنا، يتم استدعاء constructor الـ class الأب مع الأطوال
    // المقدمة لعرض وارتفاع الـ Polygon
    super(length, length);
    // ملاحظة: في الـ classes المشتقة، يجب استدعاء super() قبل
    // أن تتمكن من استخدام 'this'. حذف هذا سيسبب خطأ مرجعي.
    this.name = "Square";
  }

  get area() {
    return this.height * this.width;
  }
}
```

### الوراثة من الكائنات العادية
لا يمكن للـ classes الوراثة من الكائنات العادية (غير القابلة للإنشاء). إذا كنت تريد الوراثة من كائن عادي بجعل جميع خصائص هذا الكائن متاحة على النسخ الموروثة، يمكنك بدلاً من ذلك استخدام Object.setPrototypeOf():

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
هذا المثال يرث من كائن Date المدمج.

```js
class MyDate extends Date {
  getFormattedDate() {
    const months = [
      "Jan", "Feb", "Mar", "Apr", "May", "Jun",
      "Jul", "Aug", "Sep", "Oct", "Nov", "Dec",
    ];
    return `${this.getDate()}-${months[this.getMonth()]}-${this.getFullYear()}`;
  }
}
```

### تجنب الوراثة
الوراثة هي علاقة اقتران قوية جداً في البرمجة كائنية التوجه. وهذا يعني أن جميع سلوكيات الـ class الأساسي يتم وراثتها من قبل الـ class الفرعي بشكل افتراضي، وهو ما قد لا يكون ما تريده دائماً. على سبيل المثال، فكر في تنفيذ ReadOnlyMap:

```js
class ReadOnlyMap extends Map {
  set() {
    throw new TypeError("يجب تعيين خريطة القراءة فقط في وقت الإنشاء.");
  }
}
```

يتبين أن ReadOnlyMap غير قابل للإنشاء، لأن constructor الـ Map يستدعي طريقة set() للنسخة.

```js
const m = new ReadOnlyMap([["a", 1]]); // TypeError: يجب تعيين خريطة القراءة فقط في وقت الإنشاء.
```

قد نتجاوز هذا باستخدام علامة خاصة للإشارة إلى ما إذا كانت النسخة قيد الإنشاء. ومع ذلك، فإن المشكلة الأكبر في هذا التصميم هي أنه يخرق مبدأ استبدال Liskov، الذي ينص على أنه يجب أن يكون الـ class الفرعي قابلاً للاستبدال بالـ class الأساسي الخاص به.

بشكل عام، ما لم يكن هناك سبب وجيه جداً لاستخدام الوراثة، من الأفضل استخدام التركيب (composition) بدلاً من ذلك. التركيب يعني أن الـ class لديه مرجع لكائن من class آخر، ويستخدم هذا الكائن فقط كتفصيل تنفيذي.

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

في هذه الحالة، class ReadOnlyMap ليس class فرعياً من Map، لكنه لا يزال ينفذ معظم الطرق نفسها. هذا يعني المزيد من تكرار الكود، لكنه يعني أيضاً أن class ReadOnlyMap ليس مرتبطاً بشدة بـ class Map، ولا ينكسر بسهولة إذا تم تغيير class Map، متجنباً المشاكل الدلالية للوراثة من الكائنات المدمجة.