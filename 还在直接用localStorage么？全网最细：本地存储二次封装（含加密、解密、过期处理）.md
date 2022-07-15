# 背景

很多人在用 **localStorage** 或 **sessionStorage** 的时候喜欢直接用，明文存储，直接将信息暴露在；浏览器中，虽然一般场景下都能应付得了且简单粗暴，但特殊需求情况下，比如设置定时功能，就不能实现。就需要对其进行二次封装，为了在使用上增加些安全感，那加密也必然是少不了的了。为方便项目使用，特对常规操作进行封装。

### 结构设计

在封装一系列操作本地存储的 API 之前，先准备了一个全局对象，对具体的操作进行判断，如下：

```typescript
interface globalConfig {
  type: "localStorage" | "sessionStorage"
  prefix: string
  expire: number
  isEncrypt: boolean
}

const config: globalConfig = {
  type: "localStorage", //存储类型，localStorage | sessionStorage
  prefix: "react-view-ui_0.0.1", //版本号
  expire: 24 * 60, //过期时间，默认为一天，单位为分钟
  isEncrypt: true, //支持加密、解密数据处理
}
```

1.  **type** 表示存储类型，为 **localStorage** 或 **sessionStorage** ；
2.  **prefix** 表示视图唯一标识，如果配置可在浏览器视图中放在前缀显示；
3.  **expire** 表示过期时间，默认为一天，单位为分钟；
4.  **isEncrypt** 表示支持加密、解密数据处理；

### 加密准备工作

这里是用到了 **crypto-js** 来处理加密和解密，可先下载包并导入。

```typescript
npm i --save-dev crypto-js

import CryptoJS from 'crypto-js';
```

对 **crypto-js** 设置密钥和密钥偏移量,可以采用将一个私钥经 MD5 加密生成 16 位密钥获得。

```typescript
const SECRET_KEY = CryptoJS.enc.Utf8.parse("3333e6e143439161") //十六位十六进制数作为密钥
const SECRET_IV = CryptoJS.enc.Utf8.parse("e3bbe7e3ba84431a") //十六位十六进制数作为密钥偏移量
```

#### 加密

```typescript
const encrypt = (data: object | string): string => {
  //加密
  if (typeof data === "object") {
    try {
      data = JSON.stringify(data)
    } catch (e) {
      throw new Error("encrypt error" + e)
    }
  }
  const dataHex = CryptoJS.enc.Utf8.parse(data)
  const encrypted = CryptoJS.AES.encrypt(dataHex, SECRET_KEY, {
    iv: SECRET_IV,
    mode: CryptoJS.mode.CBC,
    padding: CryptoJS.pad.Pkcs7,
  })
  return encrypted.ciphertext.toString()
}
```

#### 解密

```typescript
const decrypt = (data: string) => {
  //解密
  const encryptedHexStr = CryptoJS.enc.Hex.parse(data)
  const str = CryptoJS.enc.Base64.stringify(encryptedHexStr)
  const decrypt = CryptoJS.AES.decrypt(str, SECRET_KEY, {
    iv: SECRET_IV,
    mode: CryptoJS.mode.CBC,
    padding: CryptoJS.pad.Pkcs7,
  })
  const decryptedStr = decrypt.toString(CryptoJS.enc.Utf8)
  return decryptedStr.toString()
}
```

这两个 API 都是将获取到的本地存储的 value 作为参数进行传递，这样就实现了加密和解密。

在传入数据进行处理、改变的时候需要进行解密；
在数据需要传出时需要进行加密。

### 核心 API 实现

#### setStorage 设置值

Storage 本身是不支持过期时间设置的，要支持设置过期时间，可以效仿 Cookie 的做法，setStorage(key, value, expire) 方法，接收三个参数，第三个参数就是设置过期时间的，用相对时间，单位分钟，要对所传参数进行类型检查。可以设置统一的过期时间，也可以对单个值得过期时间进行单独配置。

```typescript
const setStorage = (
  key: string,
  value: any,
  expire: number = 24 * 60
): boolean => {
  //设定值
  if (value === "" || value === null || value === undefined) {
    //空值重置
    value = null
  }
  if (isNaN(expire) || expire < 0) {
    //过期时间值合理性判断
    throw new Error("Expire must be a number")
  }
  const data = {
    value, //存储值
    time: Date.now(), //存储日期
    expire: Date.now() + 1000 * 60 * expire, //过期时间
  }
  //是否需要加密，判断装载加密数据或原数据
  window[config.type].setItem(
    autoAddPreFix(key),
    config.isEncrypt ? encrypt(JSON.stringify(data)) : JSON.stringify(data)
  )
  return true
}
```

#### getStorageFromKey 根据 key 获取 value

首先要对 key 是否存在进行判断，防止获取不存在的值而报错。对获取方法进一步扩展，只要在有效期内就可以获取 Storage 值，如果过期则直接删除该值，并返回 null。

```typescript
const getStorageFromKey = (key: string) => {
  //获取指定值
  if (config.prefix) {
    key = autoAddPreFix(key)
  }
  if (!window[config.type].getItem(key)) {
    //不存在判断
    return null
  }
  const storageVal = config.isEncrypt
    ? JSON.parse(decrypt(window[config.type].getItem(key) as string))
    : JSON.parse(window[config.type].getItem(key) as string)
  const now = Date.now()
  if (now >= storageVal.expire) {
    //过期销毁
    removeStorageFromKey(key)
    return null
    //不过期回值
  } else {
    return storageVal.value
  }
}
```

#### getAllStorage 获取所有存储值

```typescript
const getAllStorage = () => {
  //获取所有值
  const storageList: any = {}
  const keys = Object.keys(window[config.type])
  keys.forEach((key) => {
    const value = getStorageFromKey(key)
    if (value !== null) {
      //如果值没有过期，加入到列表中
      storageList[key] = value
    }
  })
  return storageList
}
```

#### getStorageLength 获取存储值数量

```typescript
const getStorageLength = () => {
  //获取值列表长度
  return window[config.type].length
}
```

#### removeStorageFromKey 根据 key 删除存储值

```typescript
const removeStorageFromKey = (key: string) => {
  //删除值
  if (config.prefix) {
    key = autoAddPreFix(key)
  }
  window[config.type].removeItem(key)
}
```

#### clearStorage 清空存储列表

```typescript
const clearStorage = () => {
  window[config.type].clear()
}
```

#### autoAddPreFix 基于全局配置的 prefix 参数添加前缀

```typescript
const autoAddPreFix = (key: string) => {
  //添加前缀，保持浏览器Application视图唯一性
  const prefix = config.prefix || ""
  return `${prefix}_${key}`
}
```

这是一个不导出的函数，作为整体封装的内部工具函数，在 setStorage、getStorageFromKey、removeStorageFromKey 会使用到。

#### 导出函数列表

提供了 6 个函数的处理能力，足够应对实际业务的大部分操作。

```typescript
export {
  setStorage,
  getStorageFromKey,
  getAllStorage,
  getStorageLength,
  removeStorageFromKey,
  clearStorage,
}
```

### 使用

在实际业务中使用，则将函数导入即可，这里先看下笔者的文件目录吧：
![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d90d408406cd465f82ef2846eb3c11b6~tplv-k3u1fbpfcp-zoom-1.image)
实际使用：

```typescript
import {
  setStorage,
  getStorageFromKey,
  getAllStorage,
  getStorageLength,
  removeStorageFromKey,
  clearStorage,
} from "../../_util/storage/config"

setStorage("name", "fx", 1)
setStorage("age", { now: 18 }, 100000)
setStorage("history", [1, 2, 3], 100000)
console.log(getStorageFromKey("name"))
removeStorageFromKey("name")
console.log(getStorageFromKey("name"))
console.log(getStorageLength())
console.log(getAllStorage())
clearStorage()
```

**接下来看一下浏览器视图：**

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0148f35b2eef413a9dcefb35dbbec236~tplv-k3u1fbpfcp-zoom-1.image)

可以看到，key 经过处理加入了 config.prefix 的前缀，有了唯一性。
value 经过了加密处理。

**再看一下通过 get 方式获取到的控制台值输出：**

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5adb72c4bbde4792abdf45ae504e6525~tplv-k3u1fbpfcp-zoom-1.image)

很完美，实际业务会把前缀清除返回进行处理，视图中有前缀绑定以及加密处理，保证了本地存储的安全性。

### 完整代码

config.ts:

```typescript
import { encrypt, decrypt } from "./encry"
import { globalConfig } from "./interface"

const config: globalConfig = {
  type: "localStorage", //存储类型，localStorage | sessionStorage
  prefix: "react-view-ui_0.0.1", //版本号
  expire: 24 * 60, //过期时间，默认为一天，单位为分钟
  isEncrypt: true, //支持加密、解密数据处理
}

const setStorage = (
  key: string,
  value: any,
  expire: number = 24 * 60
): boolean => {
  //设定值
  if (value === "" || value === null || value === undefined) {
    //空值重置
    value = null
  }
  if (isNaN(expire) || expire < 0) {
    //过期时间值合理性判断
    throw new Error("Expire must be a number")
  }
  const data = {
    value, //存储值
    time: Date.now(), //存储日期
    expire: Date.now() + 1000 * 60 * expire, //过期时间
  }
  //是否需要加密，判断装载加密数据或原数据
  window[config.type].setItem(
    autoAddPreFix(key),
    config.isEncrypt ? encrypt(JSON.stringify(data)) : JSON.stringify(data)
  )
  return true
}

const getStorageFromKey = (key: string) => {
  //获取指定值
  if (config.prefix) {
    key = autoAddPreFix(key)
  }
  if (!window[config.type].getItem(key)) {
    //不存在判断
    return null
  }

  const storageVal = config.isEncrypt
    ? JSON.parse(decrypt(window[config.type].getItem(key) as string))
    : JSON.parse(window[config.type].getItem(key) as string)
  const now = Date.now()
  if (now >= storageVal.expire) {
    //过期销毁
    removeStorageFromKey(key)
    return null
    //不过期回值
  } else {
    return storageVal.value
  }
}
const getAllStorage = () => {
  //获取所有值
  const storageList: any = {}
  const keys = Object.keys(window[config.type])
  keys.forEach((key) => {
    const value = getStorageFromKey(autoRemovePreFix(key))
    if (value !== null) {
      //如果值没有过期，加入到列表中
      storageList[autoRemovePreFix(key)] = value
    }
  })
  return storageList
}
const getStorageLength = () => {
  //获取值列表长度
  return window[config.type].length
}
const removeStorageFromKey = (key: string) => {
  //删除值
  if (config.prefix) {
    key = autoAddPreFix(key)
  }
  window[config.type].removeItem(key)
}
const clearStorage = () => {
  window[config.type].clear()
}
const autoAddPreFix = (key: string) => {
  //添加前缀，保持唯一性
  const prefix = config.prefix || ""
  return `${prefix}_${key}`
}
const autoRemovePreFix = (key: string) => {
  //删除前缀，进行增删改查
  const lineIndex = config.prefix.length + 1
  return key.substr(lineIndex)
}

export {
  setStorage,
  getStorageFromKey,
  getAllStorage,
  getStorageLength,
  removeStorageFromKey,
  clearStorage,
}
```

encry.ts:

```typescript
import CryptoJS from "crypto-js"

const SECRET_KEY = CryptoJS.enc.Utf8.parse("3333e6e143439161") //十六位十六进制数作为密钥
const SECRET_IV = CryptoJS.enc.Utf8.parse("e3bbe7e3ba84431a") //十六位十六进制数作为密钥偏移量

const encrypt = (data: object | string): string => {
  //加密
  if (typeof data === "object") {
    try {
      data = JSON.stringify(data)
    } catch (e) {
      throw new Error("encrypt error" + e)
    }
  }
  const dataHex = CryptoJS.enc.Utf8.parse(data)
  const encrypted = CryptoJS.AES.encrypt(dataHex, SECRET_KEY, {
    iv: SECRET_IV,
    mode: CryptoJS.mode.CBC,
    padding: CryptoJS.pad.Pkcs7,
  })
  return encrypted.ciphertext.toString()
}

const decrypt = (data: string) => {
  //解密
  const encryptedHexStr = CryptoJS.enc.Hex.parse(data)
  const str = CryptoJS.enc.Base64.stringify(encryptedHexStr)
  const decrypt = CryptoJS.AES.decrypt(str, SECRET_KEY, {
    iv: SECRET_IV,
    mode: CryptoJS.mode.CBC,
    padding: CryptoJS.pad.Pkcs7,
  })
  const decryptedStr = decrypt.toString(CryptoJS.enc.Utf8)
  return decryptedStr.toString()
}

export { encrypt, decrypt }
```

interface.ts:

```typescript
interface globalConfig {
  type: 'localStorage' | 'sessionStorage';
  prefix: string;
  expire: number;
  isEncrypt: boolean;
}

export type { globalConfig };

```

# 总结

前端开发中直接使用明文存储在本地是比较常见的一件事情同时也是不安全的一件事，对本地存储进行二次封装可以提高安全性，并且有了 API 的支持，可以在本地存储操作时更加简单。
