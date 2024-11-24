## حقول Class العامة

تُعد الحقول العامة خصائص قابلة للكتابة والتعداد والتكوين. وعلى عكس نظيراتها Private، فإنها تشارك في الوراثة النموذجية (prototype inheritance).

### التركيب النحوي
```js
class ClassWithField {
  instanceField;
  instanceFieldWithInitializer = "instance field";
  static staticField;
  static staticFieldWithInitializer = "static field";
}
```

هناك بعض القيود النحوية الإضافية:
- لا يمكن أن يكون اسم الخاصية static (سواء كانت حقلاً أو طريقة) هو prototype.
- لا يمكن أن يكون اسم حقل class (سواء كان static أو instance) هو constructor.

### الوصف

تقدم هذه الصفحة شرحاً مفصلاً لحقول instance العامة.

- للحقول static العامة، راجع static.
- للحقول private، راجع private properties.
- للطرق العامة، راجع method definitions.
- للدوال getter وsetter العامة، راجع getter وsetter.

توجد حقول instance العامة في كل instance تم إنشاؤه من class. من خلال الإعلان عن حقل عام، يمكنك ضمان وجود الحقل دائماً، ويصبح تعريف class أكثر توثيقاً لنفسه.

تتم إضافة حقول instance العامة إلى instance إما في وقت الإنشاء في class الأساسي (قبل تنفيذ جسم constructor)، أو مباشرة بعد عودة super() في subclass. الحقول بدون مُهيئات (initializers) يتم تهيئتها بقيمة undefined. مثل الخصائص، يمكن أن تكون أسماء الحقول محسوبة (computed).

```js
const PREFIX = "prefix";

class ClassWithField {
  field;
  fieldWithInitializer = "instance field";
  [`${PREFIX}Field`] = "prefixed field";
}

const instance = new ClassWithField();
console.log(Object.hasOwn(instance, "field")); // true
console.log(instance.field); // undefined
console.log(instance.fieldWithInitializer); // "instance field"
console.log(instance.prefixField); // "prefixed field"
```

يتم تقييم أسماء الحقول المحسوبة مرة واحدة فقط، في وقت تعريف class. هذا يعني أن كل class يحتوي دائماً على مجموعة ثابتة من أسماء الحقول، ولا يمكن أن يكون لـ instances مختلفة أسماء حقول مختلفة عبر الأسماء المحسوبة. قيمة this في التعبير المحسوب هي this المحيطة بتعريف class، والإشارة إلى اسم class تؤدي إلى ReferenceError لأن class لم يتم تهيئته بعد. تعمل await وyield كما هو متوقع في هذا التعبير.

```js
class C {
  [Math.random()] = 1;
}

console.log(new C());
console.log(new C());
// كلا الـ instances لديهما نفس اسم الحقل
```

في مُهيئ الحقل، تشير this إلى instance الـ class قيد الإنشاء، وتشير super إلى خاصية prototype للـ class الأساسي، والتي تحتوي على طرق instance للـ class الأساسي، ولكن ليس حقول instance الخاصة به.

```js
class Base {
  baseField = "base field";
  anotherBaseField = this.baseField;
  baseMethod() {
    return "base method output";
  }
}

class Derived extends Base {
  subField = super.baseMethod();
}

const base = new Base();
const sub = new Derived();

console.log(base.anotherBaseField); // "base field"
console.log(sub.subField); // "base method output"
```

يتم تقييم تعبير مُهيئ الحقل في كل مرة يتم فيها إنشاء instance جديد. (لأن قيمة this مختلفة لكل instance، يمكن لتعبير المُهيئ الوصول إلى الخصائص الخاصة بكل instance.)

```js
class C {
  obj = {};
}

const instance1 = new C();
const instance2 = new C();
console.log(instance1.obj === instance2.obj); // false
```

يتم تقييم التعبير بشكل متزامن. لا يمكنك استخدام await أو yield في تعبير المُهيئ. (فكر في تعبير المُهيئ كما لو كان ملفوفاً ضمنياً في دالة.)

نظراً لأن حقول instance الخاصة بـ class تتم إضافتها قبل تشغيل constructor المعني، يمكنك الوصول إلى قيم الحقول داخل constructor. ومع ذلك، نظراً لأن حقول instance الخاصة بـ class المشتق يتم تعريفها بعد عودة super()، فإن constructor الخاص بـ class الأساسي لا يمكنه الوصول إلى حقول class المشتق.

```js
class Base {
  constructor() {
    console.log("Base constructor:", this.field);
  }
}

class Derived extends Base {
  field = 1;
  constructor() {
    super();
    console.log("Derived constructor:", this.field);
    this.field = 2;
  }
}

const instance = new Derived();
// Base constructor: undefined
// Derived constructor: 1
console.log(instance.field); // 2
```

تتم إضافة الحقول واحداً تلو الآخر. يمكن لمُهيئات الحقول الإشارة إلى قيم الحقول فوقها، ولكن ليس أسفلها. تتم إضافة جميع طرق instance وstatic مسبقاً ويمكن الوصول إليها، على الرغم من أن استدعاءها قد لا يتصرف كما هو متوقع إذا كانت تشير إلى حقول أسفل الحقل الذي تتم تهيئته.

```js
class C {
  a = 1;
  b = this.c;
  c = this.a + 1;
  d = this.c + 1;
}

const instance = new C();
console.log(instance.d); // 3
console.log(instance.b); // undefined
```

ملاحظة: هذا أكثر أهمية مع الحقول private، لأن الوصول إلى حقل private غير مُهيأ يؤدي إلى TypeError، حتى لو تم الإعلان عن الحقل private أدناه. (إذا لم يتم الإعلان عن الحقل private، فسيكون SyntaxError مبكراً.)

نظراً لأن حقول class تتم إضافتها باستخدام دلالة [[DefineOwnProperty]] (وهي أساساً Object.defineProperty())، فإن إعلانات الحقول في classes المشتقة لا تستدعي setters في class الأساسي. يختلف هذا السلوك عن استخدام this.field = ... في constructor.

```js
class Base {
  set field(val) {
    console.log(val);
  }
}

class DerivedWithField extends Base {
  field = 1;
}

const instance = new DerivedWithField(); // لا يوجد سجل

class DerivedWithConstructor extends Base {
  constructor() {
    super();
    this.field = 1;
  }
}

const instance2 = new DerivedWithConstructor(); // يسجل 1
```

ملاحظة: قبل الانتهاء من مواصفات حقول class مع دلالة [[DefineOwnProperty]]، كانت معظم المترجمات، بما في ذلك Babel وtsc، تحول حقول class إلى شكل DerivedWithConstructor، مما تسبب في أخطاء دقيقة بعد توحيد معايير حقول class.

### أمثلة

#### استخدام حقول Class
لا يمكن أن تعتمد حقول class على وسائط constructor، لذا عادةً ما تقيّم مُهيئات الحقول إلى نفس القيمة لكل instance (ما لم يكن نفس التعبير قادراً على تقييم قيم مختلفة في كل مرة، مثل Date.now() أو مُهيئات الكائنات).

```js
class Person {
  name = nameArg; // nameArg خارج نطاق constructor
  constructor(nameArg) {}
}
```

```js
class Person {
  // جميع instances من Person سيكون لديها نفس الاسم
  name = "Dragomir";
}
```

ومع ذلك، حتى الإعلان عن حقل class فارغ مفيد، لأنه يشير إلى وجود الحقل، مما يسمح لمدققي النوع وكذلك القراء البشريين بتحليل شكل class بشكل ثابت.

```js
class Person {
  name;
  age;
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
}
```

يبدو الكود أعلاه مكرراً، ولكن فكر في الحالة التي يتم فيها تغيير this ديناميكياً: الإعلان الصريح عن الحقل يوضح أي الحقول ستكون موجودة بالتأكيد في instance.

```js
class Person {
  name;
  age;
  constructor(properties) {
    Object.assign(this, properties);
  }
}
```

نظراً لأن المُهيئات يتم تقييمها بعد تنفيذ class الأساسي، يمكنك الوصول إلى الخصائص التي تم إنشاؤها بواسطة constructor الـ class الأساسي.

```js
class Person {
  name;
  age;
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
}

class Professor extends Person {
  name = `Professor ${this.name}`;
}

console.log(new Professor("Radev", 54).name); // "Professor Radev"
```