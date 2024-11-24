# try...catch
متوفر على نطاق واسع كخط أساسي

تتكون عبارة try...catch من كتلة try وإما كتلة catch أو كتلة finally أو كليهما. يتم تنفيذ الكود في كتلة try أولاً، وإذا حدث استثناء (exception)، سيتم تنفيذ الكود في كتلة catch. أما الكود في كتلة finally فسيتم تنفيذه دائماً قبل الخروج من البنية بأكملها.

## بناء الجملة (Syntax)
```js
try {
  tryStatements
} catch (exceptionVar) {
  catchStatements
} finally {
  finallyStatements
}
```

### tryStatements
العبارات المراد تنفيذها.

### catchStatements
العبارات التي يتم تنفيذها إذا تم إلقاء استثناء في كتلة try.

### exceptionVar (اختياري)
معرّف أو نمط اختياري لحمل الاستثناء الملتقط للكتلة catch المرتبطة. إذا كانت كتلة catch لا تستخدم قيمة الاستثناء، يمكنك حذف exceptionVar والأقواس المحيطة به.

### finallyStatements
العبارات التي يتم تنفيذها قبل خروج تدفق التحكم من بنية try...catch...finally. يتم تنفيذ هذه العبارات بغض النظر عما إذا تم إلقاء استثناء أو التقاطه.

## الوصف
تبدأ عبارة try دائماً بكتلة try. ثم يجب أن تكون هناك كتلة catch أو كتلة finally. من الممكن أيضاً وجود كتلتي catch و finally معاً. هذا يعطينا ثلاثة أشكال لعبارة try:

- try...catch
- try...finally
- try...catch...finally

على عكس البنى الأخرى مثل if أو for، يجب أن تكون كتل try و catch و finally كتلاً كاملة، وليست عبارات مفردة.

```js
try doSomething(); // SyntaxError
catch (e) console.log(e);
```

تحتوي كتلة catch على عبارات تحدد ما يجب فعله إذا تم إلقاء استثناء في كتلة try. إذا ألقت أي عبارة داخل كتلة try (أو في دالة تم استدعاؤها من داخل كتلة try) استثناءً، يتم نقل التحكم فوراً إلى كتلة catch. إذا لم يتم إلقاء أي استثناء في كتلة try، يتم تخطي كتلة catch.

سيتم تنفيذ كتلة finally دائماً قبل خروج تدفق التحكم من بنية try...catch...finally. يتم تنفيذها دائماً، بغض النظر عما إذا تم إلقاء استثناء أو التقاطه.

يمكنك تداخل عبارة try واحدة أو أكثر. إذا لم تحتوي عبارة try الداخلية على كتلة catch، يتم استخدام كتلة catch الخاصة بعبارة try المحيطة بدلاً من ذلك.

يمكنك أيضاً استخدام عبارة try للتعامل مع استثناءات JavaScript. راجع دليل JavaScript للحصول على مزيد من المعلومات حول استثناءات JavaScript.

## ربط Catch
عندما يتم إلقاء استثناء في كتلة try، يحمل exceptionVar (مثل e في catch (e)) قيمة الاستثناء. يمكنك استخدام هذا الربط للحصول على معلومات حول الاستثناء الذي تم إلقاؤه. هذا الربط متاح فقط في نطاق كتلة catch.

لا يحتاج أن يكون معرفاً واحداً. يمكنك استخدام نمط التفكيك لتعيين معرفات متعددة دفعة واحدة:

```js
try {
  throw new TypeError("oops");
} catch ({ name, message }) {
  console.log(name); // "TypeError"
  console.log(message); // "oops"
}
```

الروابط التي تم إنشاؤها بواسطة عبارة catch تعيش في نفس نطاق كتلة catch، لذا لا يمكن للمتغيرات المعلنة في كتلة catch أن تحمل نفس اسم الروابط التي تم إنشاؤها بواسطة عبارة catch.

```js
try {
  throw new TypeError("oops");
} catch ({ name, message }) {
  var name; // SyntaxError: Identifier 'name' has already been declared
  let message; // SyntaxError: Identifier 'message' has already been declared
}
```

ربط الاستثناء قابل للكتابة. على سبيل المثال، قد ترغب في توحيد قيمة الاستثناء للتأكد من أنه كائن Error:

```js
try {
  throw "Oops; this is not an Error object";
} catch (e) {
  if (!(e instanceof Error)) {
    e = new Error(e);
  }
  console.error(e.message);
}
```

إذا كنت لا تحتاج إلى قيمة الاستثناء، يمكنك حذفها مع الأقواس المحيطة بها:

```js
function isValidJSON(text) {
  try {
    JSON.parse(text);
    return true;
  } catch {
    return false;
  }
}
```

## كتلة finally
تحتوي كتلة finally على عبارات يتم تنفيذها بعد تنفيذ كتلة try وكتلة catch، ولكن قبل العبارات التي تلي بنية try...catch...finally. سيدخل تدفق التحكم دائماً إلى كتلة finally، والتي يمكن أن تتم بإحدى الطرق التالية:

- مباشرة بعد انتهاء تنفيذ كتلة try بشكل طبيعي (ولم يتم إلقاء أي استثناءات)
- مباشرة بعد انتهاء تنفيذ كتلة catch بشكل طبيعي
- مباشرة قبل تنفيذ عبارة تحكم بالتدفق (return, throw, break, continue) في كتلة try أو catch

إذا تم إلقاء استثناء من كتلة try، حتى عندما لا توجد كتلة catch للتعامل مع الاستثناء، سيتم تنفيذ كتلة finally، وفي هذه الحالة يتم إلقاء الاستثناء مباشرة بعد انتهاء تنفيذ كتلة finally.

المثال التالي يوضح حالة استخدام لكتلة finally. يفتح الكود ملفاً ثم ينفذ عبارات تستخدم الملف؛ تضمن كتلة finally إغلاق الملف دائماً بعد استخدامه حتى لو تم إلقاء استثناء:

```js
openMyFile();
try {
  // tie up a resource
  writeMyFile(theData);
} finally {
  closeMyFile(); // always close the resource
}
```

عبارات تدفق التحكم (return, throw, break, continue) في كتلة finally ستقوم "بإخفاء" أي قيمة إكمال من كتلة try أو catch. في هذا المثال، تحاول كتلة try إرجاع 1، ولكن قبل الإرجاع، يتم تسليم تدفق التحكم إلى كتلة finally أولاً، لذا يتم إرجاع قيمة العودة من كتلة finally بدلاً من ذلك:

```js
function doIt() {
  try {
    return 1;
  } finally {
    return 2;
  }
}

doIt(); // returns 2
```

من غير المستحسن عموماً وجود عبارات تدفق التحكم في كتلة finally. استخدمها فقط لكود التنظيف.

## أمثلة
### كتلة catch غير مشروطة
عندما يتم استخدام كتلة catch، يتم تنفيذها عند إلقاء أي استثناء من داخل كتلة try:

```js
try {
  throw "myException"; // generates an exception
} catch (e) {
  // statements to handle any exceptions
  logMyErrors(e); // pass exception object to error handler
}
```

تحدد كتلة catch معرفاً (e في المثال أعلاه) الذي يحمل قيمة الاستثناء؛ هذه القيمة متاحة فقط في نطاق كتلة catch.

### كتل catch المشروطة
يمكنك إنشاء "كتل catch مشروطة" عن طريق دمج كتل try...catch مع بنى if...else if...else:

```js
try {
  myRoutine(); // may throw three types of exceptions
} catch (e) {
  if (e instanceof TypeError) {
    // statements to handle TypeError exceptions
  } else if (e instanceof RangeError) {
    // statements to handle RangeError exceptions
  } else if (e instanceof EvalError) {
    // statements to handle EvalError exceptions
  } else {
    // statements to handle any unspecified exceptions
    logMyErrors(e); // pass exception object to error handler
  }
}
```

حالة استخدام شائعة لهذا هي التقاط (وإسكات) مجموعة فرعية صغيرة من الأخطاء المتوقعة فقط، ثم إعادة إلقاء الخطأ في الحالات الأخرى:

```js
try {
  myRoutine();
} catch (e) {
  if (e instanceof RangeError) {
    // statements to handle this very common expected error
  } else {
    throw e; // re-throw the error unchanged
  }
}
```

هذا قد يحاكي بناء الجملة من لغات أخرى، مثل Java:

```java
try {
  myRoutine();
} catch (RangeError e) {
  // statements to handle this very common expected error
}
// Other errors are implicitly re-thrown
```

### كتل try المتداخلة
أولاً، دعونا نرى ماذا يحدث مع هذا:

```js
try {
  try {
    throw new Error("oops");
  } finally {
    console.log("finally");
  }
} catch (ex) {
  console.error("outer", ex.message);
}

// Logs:
// "finally"
// "outer" "oops"
```

الآن، إذا قمنا بالفعل بالتقاط الاستثناء في كتلة try الداخلية عن طريق إضافة كتلة catch:

```js
try {
  try {
    throw new Error("oops");
  } catch (ex) {
    console.error("inner", ex.message);
  } finally {
    console.log("finally");
  }
} catch (ex) {
  console.error("outer", ex.message);
}

// Logs:
// "inner" "oops"
// "finally"
```

والآن، دعونا نعيد إلقاء الخطأ:

```js
try {
  try {
    throw new Error("oops");
  } catch (ex) {
    console.error("inner", ex.message);
    throw ex;
  } finally {
    console.log("finally");
  }
} catch (ex) {
  console.error("outer", ex.message);
}

// Logs:
// "inner" "oops"
// "finally"
// "outer" "oops"
```

سيتم التقاط أي استثناء معين مرة واحدة فقط بواسطة أقرب كتلة catch محيطة ما لم يتم إعادة إلقائه. بالطبع، أي استثناءات جديدة تنشأ في الكتلة "الداخلية" (لأن الكود في كتلة catch قد يفعل شيئاً يؤدي إلى إلقاء استثناء) سيتم التقاطها بواسطة الكتلة "الخارجية".

### العودة من كتلة finally
إذا أعادت كتلة finally قيمة، تصبح هذه القيمة هي قيمة العودة لعبارة try-catch-finally بأكملها، بغض النظر عن أي عبارات return في كتل try و catch. وهذا يشمل الاستثناءات التي تم إلقاؤها داخل كتلة catch:

```js
(() => {
  try {
    try {
      throw new Error("oops");
    } catch (ex) {
      console.error("inner", ex.message);
      throw ex;
    } finally {
      console.log("finally");
      return;
    }
  } catch (ex) {
    console.error("outer", ex.message);
  }
})();

// Logs:
// "inner" "oops"
// "finally"
```

لا يتم إلقاء "oops" الخارجي بسبب return في كتلة finally. نفس الشيء ينطبق على أي قيمة يتم إرجاعها من كتلة catch.