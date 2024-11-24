## الخصائص Private

تُعد الخصائص Private نظيرة للخصائص العادية للclass والتي تكون public، بما في ذلك حقول class وطرق class وغيرها. يتم إنشاء الخصائص Private باستخدام بادئة الرمز # ولا يمكن الوصول إليها قانونياً خارج نطاق class. يتم فرض خصوصية هذه الخصائص بواسطة JavaScript نفسها. الطريقة الوحيدة للوصول إلى خاصية private هي عبر علامة النقطة (dot notation)، ويمكن القيام بذلك فقط داخل class الذي يُعرّف الخاصية private.

لم تكن الخصائص private أصيلة في اللغة قبل وجود هذا التركيب النحوي. في الوراثة النموذجية (prototypal inheritance)، يمكن محاكاة سلوكها باستخدام كائنات WeakMap أو closures، لكنها لا تقارن بالتركيب النحوي من حيث سهولة الاستخدام.

### التركيب النحوي
```js
class ClassWithPrivate {
  #privateField;
  #privateFieldWithInitializer = 42;

  #privateMethod() {
    // ...
  }

  static #privateStaticField;
  static #privateStaticFieldWithInitializer = 42;

  static #privateStaticMethod() {
    // ...
  }
}
```

هناك بعض القيود النحوية الإضافية:
- يجب أن تكون جميع المعرّفات private المُعلنة داخل class فريدة. تتم مشاركة نطاق الأسماء بين الخصائص static والinstance. الاستثناء الوحيد هو عندما يُعرّف الإعلانان زوجاً من getter-setter.
- لا يمكن أن يكون المعرّف private هو #constructor.

### الوصف

معظم خصائص class لها نظائر private:
- الحقول Private
- الطرق Private
- الحقول Static Private
- الطرق Static Private
- الدوال Getter Private
- الدوال Setter Private
- الدوال Static Getter Private
- الدوال Static Setter Private

يُطلق على هذه الميزات مجتمعة اسم "الخصائص Private". ومع ذلك، لا يمكن أن تكون constructors خاصة في JavaScript. لمنع إنشاء classes خارج نطاق class، يجب استخدام علامة private.

يتم الإعلان عن الخصائص Private باستخدام أسماء تبدأ بـ # (تُنطق "hash names")، وهي معرّفات مسبوقة بـ #. البادئة # هي جزء أساسي من اسم الخاصية - يمكنك رؤية العلاقة مع اتفاقية البادئة القديمة _privateField - لكنها ليست خاصية نصية عادية، لذا لا يمكنك الوصول إليها ديناميكياً باستخدام علامات الأقواس.

يُعد خطأً نحوياً الإشارة إلى أسماء # من خارج class. كما أنه خطأ نحوي الإشارة إلى خصائص private لم يتم الإعلان عنها في جسم class، أو محاولة إزالة الخصائص المُعلنة باستخدام delete.

### أمثلة

#### الحقول Private
تشمل الحقول Private حقول instance private وحقول static private. يمكن الوصول إلى الحقول Private فقط من داخل إعلان class.

##### حقول Instance Private
مثل نظيراتها العامة، فإن حقول instance private:
- تتم إضافتها قبل تشغيل constructor في class الأساسي، أو مباشرة بعد استدعاء super() في subclass
- تكون متاحة فقط على instances من class

```js
class ClassWithPrivateField {
  #privateField;

  constructor() {
    this.#privateField = 42;
  }
}

class Subclass extends ClassWithPrivateField {
  #subPrivateField;

  constructor() {
    super();
    this.#subPrivateField = 23;
  }
}

new Subclass(); // في بعض أدوات التطوير، يظهر Subclass {#privateField: 42, #subPrivateField: 23}
```

ملاحظة: #privateField من class الأساسي ClassWithPrivateField خاص بـ ClassWithPrivateField ولا يمكن الوصول إليه من Subclass المشتق.

##### إعادة كائن مُتجاوز
يمكن لـ constructor الخاص بـ class أن يُعيد كائناً مختلفاً، والذي سيُستخدم كـ this الجديد لـ constructor الفئة المشتقة. يمكن للفئة المشتقة بعد ذلك تعريف حقول private على ذلك الكائن المُعاد - مما يعني أنه من الممكن "وضع" حقول private على كائنات غير مرتبطة.

```js
class Stamper extends class {
  // فئة أساسية يُعيد constructor الخاص بها الكائن الذي يتلقاه
  constructor(obj) {
    return obj;
  }
} {
  // هذا الإعلان سيضع الحقل private على الكائن
  // المُعاد من constructor الفئة الأساسية
  #stamp = 42;
  static getStamp(obj) {
    return obj.#stamp;
  }
}

const obj = {};
new Stamper(obj);
// يستدعي Stamper الفئة Base، التي تُعيد obj، لذا يصبح obj هو
// قيمة this. ثم يُعرّف Stamper الحقل #stamp على obj

console.log(obj); // في بعض أدوات التطوير، يظهر {#stamp: 42}
console.log(Stamper.getStamp(obj)); // 42
console.log(obj instanceof Stamper); // false

// لا يمكنك وضع خصائص private مرتين
new Stamper(obj); // خطأ: تهيئة كائن مرتين تُعد خطأً مع الحقول private
```

تحذير: هذا أمر قد يكون مربكاً للغاية. يُنصح عموماً بتجنب إعادة أي شيء من constructor - خاصةً إذا كان غير مرتبط بـ this.

##### الحقول Static Private
مثل نظيراتها العامة، فإن الحقول static private:
- تتم إضافتها إلى constructor الخاص بـ class في وقت تقييم class
- تكون متاحة فقط على class نفسه

```js
class ClassWithPrivateStaticField {
  static #privateStaticField = 42;

  static publicStaticMethod() {
    return ClassWithPrivateStaticField.#privateStaticField;
  }
}

console.log(ClassWithPrivateStaticField.publicStaticMethod()); // 42
```

هناك قيد على الحقول static private: فقط class الذي يُعرّف الحقل static private يمكنه الوصول إلى الحقل. يمكن أن يؤدي هذا إلى سلوك غير متوقع عند استخدام this. في المثال التالي، يشير this إلى class الـ Subclass (وليس class الـ ClassWithPrivateStaticField) عندما نحاول استدعاء Subclass.publicStaticMethod()، وبالتالي يتسبب في TypeError.

```js
class ClassWithPrivateStaticField {
  static #privateStaticField = 42;

  static publicStaticMethod() {
    return this.#privateStaticField;
  }
}

class Subclass extends ClassWithPrivateStaticField {}

Subclass.publicStaticMethod(); // TypeError: لا يمكن قراءة العضو الخاص #privateStaticField من كائن لم يُعلن class الخاص به عنه
```

ينطبق نفس الأمر إذا استدعيت الطريقة باستخدام super، لأن طرق super لا يتم استدعاؤها مع super class كـ this.

```js
class ClassWithPrivateStaticField {
  static #privateStaticField = 42;

  static publicStaticMethod() {
    // عند الاستدعاء من خلال super، يظل this يشير إلى Subclass
    return this.#privateStaticField;
  }
}

class Subclass extends ClassWithPrivateStaticField {
  static callSuperMethod() {
    return super.publicStaticMethod();
  }
}

Subclass.callSuperMethod(); // TypeError: لا يمكن قراءة العضو الخاص #privateStaticField من كائن لم يُعلن class الخاص به عنه
```

يُنصح دائماً بالوصول إلى الحقول static private من خلال اسم class، وليس من خلال this، حتى لا تنكسر الطريقة عند الوراثة.

#### الطرق Private
تشمل الطرق Private طرق instance private وطرق static private. يمكن الوصول إلى الطرق Private فقط من داخل إعلان class.

##### طرق Instance Private
على عكس نظيراتها العامة، فإن طرق instance private:
- يتم تثبيتها مباشرة قبل تثبيت حقول instance
- تكون متاحة فقط على instances من class، وليس على خاصية prototype الخاصة بها

```js
class ClassWithPrivateMethod {
  #privateMethod() {
    return 42;
  }

  publicMethod() {
    return this.#privateMethod();
  }
}

const instance = new ClassWithPrivateMethod();
console.log(instance.publicMethod()); // 42
```

يمكن أن تكون طرق instance private دوال generator، أو async، أو async generator. كما أن الدوال getter وsetter الخاصة ممكنة أيضاً، وتتبع نفس متطلبات التركيب النحوي مثل نظيراتها العامة من getter وsetter.

```js
class ClassWithPrivateAccessor {
  #message;

  get #decoratedMessage() {
    return `🎬${this.#message}🛑`;
  }
  set #decoratedMessage(msg) {
    this.#message = msg;
  }

  constructor() {
    this.#decoratedMessage = "hello world";
    console.log(this.#decoratedMessage);
  }
}

new ClassWithPrivateAccessor(); // 🎬hello world🛑
```

على عكس الطرق العامة، لا يمكن الوصول إلى الطرق private على خاصية prototype الخاصة بـ class الخاص بها.

```js
class C {
  #method() {}

  static getMethod(x) {
    return x.#method;
  }
}

console.log(C.getMethod(new C())); // [Function: #method]
console.log(C.getMethod(C.prototype)); // TypeError: يجب أن يكون المستقبِل instance من class C
```

##### الطرق Static Private
مثل نظيراتها العامة، فإن الطرق static private:
- تتم إضافتها إلى constructor الخاص بـ class في وقت تقييم class
- تكون متاحة فقط على class نفسه

```js
class ClassWithPrivateStaticMethod {
  static #privateStaticMethod() {
    return 42;
  }

  static publicStaticMethod() {
    return ClassWithPrivateStaticMethod.#privateStaticMethod();
  }
}

console.log(ClassWithPrivateStaticMethod.publicStaticMethod()); // 42
```

يمكن أن تكون الطرق static private دوال generator، أو async، أو async generator.

ينطبق نفس القيد المذكور سابقاً للحقول static private على الطرق static private، ويمكن أن يؤدي بالمثل إلى سلوك غير متوقع عند استخدام this. في المثال التالي، عندما نحاول استدعاء Subclass.publicStaticMethod()، يشير this إلى class الـ Subclass (وليس class الـ ClassWithPrivateStaticMethod) وبالتالي يتسبب في TypeError.

```js
class ClassWithPrivateStaticMethod {
  static #privateStaticMethod() {
    return 42;
  }

  static publicStaticMethod() {
    return this.#privateStaticMethod();
  }
}

class Subclass extends ClassWithPrivateStaticMethod {}

console.log(Subclass.publicStaticMethod()); // TypeError: لا يمكن قراءة العضو الخاص #privateStaticMethod من كائن لم يُعلن class الخاص به عنه
```

#### محاكاة Constructors الخاصة
تتضمن العديد من اللغات الأخرى القدرة على تمييز constructor كـ private، مما يمنع إنشاء instances من class خارج class نفسه - يمكنك فقط استخدام طرق factory ثابتة تقوم بإنشاء instances، أو عدم القدرة على إنشاء instances على الإطلاق. لا تمتلك JavaScript طريقة أصلية للقيام بذلك، ولكن يمكن تحقيقه باستخدام علامة static private.

```js
class PrivateConstructor {
  static #isInternalConstructing = false;

  constructor() {
    if (!PrivateConstructor.#isInternalConstructing) {
      throw new TypeError("لا يمكن إنشاء PrivateConstructor");
    }
    PrivateConstructor.#isInternalConstructing = false;
    // المزيد من منطق التهيئة
  }

  static create() {
    PrivateConstructor.#isInternalConstructing = true;
    const instance = new PrivateConstructor();
    return instance;
  }
}

new PrivateConstructor(); // TypeError: لا يمكن إنشاء PrivateConstructor
PrivateConstructor.create(); // PrivateConstructor {}
```