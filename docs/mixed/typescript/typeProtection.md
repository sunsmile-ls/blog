#### ts 的类型保护

```typescript
enum Type {
  Strong,
  Week
}

class Java {
  helloJava() {
    console.log('Hello Java')
  }
  java: any
}

class JavaScript {
  helloJavaScript() {
    console.log('Hello JavaScript')
  }
  js: any
}

function isJava(lang: Java | JavaScript): lang is Java {
  return (lang as Java).helloJava !== undefined
}

function getLanguage(type: Type, x: string | number) {
  let lang = type === Type.Strong ? new Java() : new JavaScript()

  if (isJava(lang)) {
    lang.helloJava()
  } else {
    lang.helloJavaScript()
  }

  // if ((lang as Java).helloJava) {
  //     (lang as Java).helloJava();
  // } else {
  //     (lang as JavaScript).helloJavaScript();
  // }

  // 1) instanceof
  // if (lang instanceof Java) {
  //     lang.helloJava()
  //     // lang.helloJavaScript()
  // } else {
  //     lang.helloJavaScript()
  // }

  // 2) in 对于属性的判断
  // if ('java' in lang) {
  //     lang.helloJava()
  // } else {
  //     lang.helloJavaScript()
  // }

  // 3) typeof
  // if (typeof x === 'string') {
  //     console.log(x.length)
  // } else {
  //     console.log(x.toFixed(2))
  // }

  return lang
}

getLanguage(Type.Week, 1)
```
