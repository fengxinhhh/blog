# 前言

博主最近在做组件库封装开发的工作，Form 表单比较复杂，包含非受控表单和受控表单，特此记录一下。

# 组件完成页面

![请添加图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/617fe21fd7fe4673b25490fbc7be7d59~tplv-k3u1fbpfcp-zoom-1.image)

封装好的 Form 表单涵盖了布局、非受控、受控、校验、重置等功能。

# 非受控表单

首先我们先做一个架子，也就是简单的非受控表单，也就是生成基础布局，不做表单内容（状态）的处理，我是一共写了两个组件，分别为 Form 和 Form.Item，基本的使用代码是这样的：

```typescript
<Form layout={"vertical"} style={{ width: "600px" }}>
  <Form.Item label="Username">
    <Input placeholder="Please enter your usename" width="200"></Input>
  </Form.Item>
  <Form.Item label="Post">
    <Input placeholder="Please enter your post" width="200"></Input>
  </Form.Item>
  <Form.Item wrapperTol={20}>
    <CheckBox checked={true}>I have read the manual</CheckBox>
  </Form.Item>
  <Form.Item wrapperTol={5}>
    <Button type="primary">Submit</Button>
  </Form.Item>
</Form>
```

效果是这样的：

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/347bc489d1d94d1c92ba544f272ff128~tplv-k3u1fbpfcp-zoom-1.image)

其实 props 也很好分了，Form 的 props 用来做整体的一些控制，如这里的 layout 就是布局，以及 style 整体样式；Form.Item 的 props 则对单行做处理。

先看一下最基本的架子吧：

Form.tsx:

```typescript
return (
  <ctx.Provider value={providerList}>
    <div className="form" style={style} ref={formField || null}>
      {disabled && <div className="disabled" />}
      {children}
    </div>
  </ctx.Provider>
)
```

Form 把所有内部 Dom 渲染出来，并且把 form 的 props 通过 react.createContext 传递给所有的 Form.Item，仅此而已。

FormItem.tsx:

```typescript
return (
  <div className="form-item" style={propsStyle}>
    <div className="label" style={labelStyle}>
      {rules.length > 0 && (
        <svg
          fill="currentColor"
          viewBox="0 0 1024 1024"
          width="0.5em"
          height="0.5em"
        >
          <path d="M583.338667 17.066667c18.773333 0 34.133333 15.36 34.133333 34.133333v349.013333l313.344-101.888a34.133333 34.133333 0 0 1 43.008 22.016l42.154667 129.706667a34.133333 34.133333 0 0 1-21.845334 43.178667l-315.733333 102.4 208.896 287.744a34.133333 34.133333 0 0 1-7.509333 47.786666l-110.421334 80.213334a34.133333 34.133333 0 0 1-47.786666-7.509334L505.685333 706.218667 288.426667 1005.226667a34.133333 34.133333 0 0 1-47.786667 7.509333l-110.421333-80.213333a34.133333 34.133333 0 0 1-7.509334-47.786667l214.186667-295.253333L29.013333 489.813333a34.133333 34.133333 0 0 1-22.016-43.008l42.154667-129.877333a34.133333 34.133333 0 0 1 43.008-22.016l320.512 104.106667L412.672 51.2c0-18.773333 15.36-34.133333 34.133333-34.133333h136.533334z"></path>
        </svg>
      )}
      {label || ""}
    </div>
    <div
      className={field || "content"}
      style={Ctx.get("layout") === "horizontal" ? { position: "relative" } : {}}
    >
      {children}
      {disabled && <div className="form-item-disabled"></div>}
      {field && rules.length > 0 && (
        <div className="hide-rule-label">{rules[0].message}</div>
      )}
    </div>
  </div>
)
```

可以看到，FormItem 主要是根据 rules、disabled 在做处理，field 可以先不看，这是受控表单相关的 props，后面会说到。

```typescript
const FormItem = (props: FormItemProps) => {
  const {
    children,
    style = {},
    label,
    wrapperCol = 0,
    wrapperTol = 0,
    field,
    rules = [],
    disabled = false,
  } = props;

  const [propsStyle, setPropsStyle] = useState({});
  const [labelStyle, setLabelStyle] = useState({});

  const Ctx = (function () {
    //创建一个ctx单例，防止组件内污染全局变量
    const c = useContext(ctx);
    return {
      get: (prop: string) => {
        return c[prop] || null;
      },
    };
  })();

  useEffect(() => {
    setPropsStyle({ ...getPropsStyles(), ...style });
    setLabelStyle(getLabelPropsStyle());
  }, [props]);

  const getPropsStyles = useCallback(() => {
    //基于props，动态构建一个props style集合
    const formAttrs = new FormItemAttrs(wrapperCol, wrapperTol, Ctx.get('layout'));
    return formAttrs.getStyle();
  }, [wrapperCol, wrapperTol, Ctx.get('layout')]);
  const getLabelPropsStyle = useCallback(() => {
    //基于props，动态构建一个label props style集合
    const labelAttrs = new FormItemLabel(Ctx.get('layout'));
    return labelAttrs.getStyle();
  }, [Ctx.get('layout')]);

.......

}
```

逻辑部分的代码这是里使用 react.useContext 接受了 Form 的全局参数，因为一些布局相关是需要每个 Item 去配合的，所以这里写了两个 useMemo 的样式方法，内部的 new FormItemAttrsz 则是把样式相关的 props 传给一个类，具体的类是这样写的：

```typescript
class FormItemAttrs {
  wrapperCol: number //底部距离
  wrapperTol: number //顶部距离
  layout: string //表单布局形式

  constructor(wrapperCol: number, wrapperTol: number, layout: string) {
    this.wrapperCol = wrapperCol
    this.wrapperTol = wrapperTol
    this.layout = layout
  }
  getStyle() {
    return {
      marginBottom: `${20 + this.wrapperCol}px`,
      marginTop: `${20 + this.wrapperTol}px`,
      ...this.formatLayout(),
    }
  }
  formatLayout() {
    let layoutStyle = {}
    switch (this.layout) {
      case "horizontal":
        layoutStyle = {}
        break
      case "vertical":
        layoutStyle = {
          flexDirection: "column",
          alignItems: "flex-start",
        }
        break
    }
    return layoutStyle
  }
}
```

基于传来的各种 props，做一个整体样式汇总，最后返回。
然后给 Form 加一个 Item 属性，内容就是 Item 组件：

```typescript
import FormItem from './form-item';

...

Form.Item = FormItem;

export default Form;
```

就这样，一个简单的非受控表单就完成了，这样就会有一个问题，用户使用，每个 FormItem 的值只能用户自己控制，就像这样

```typescript
export default function index1() {
  const [username, setUsername] = useState("小明")
  const changeVal = (e) => {
    setUsername(e.target.value)
  }

  return (
    <div>
      <Form layout={"vertical"} style={{ width: "600px" }}>
        <Form.Item label="Username">
          <Input
            placeholder="Please enter your usename"
            width="200"
            value={username}
            onChange={(e) => changeVal(e)}
          ></Input>
        </Form.Item>
        <Form.Item label="Post">
          <Input placeholder="Please enter your post" width="200"></Input>
        </Form.Item>
        <Form.Item wrapperTol={20}>
          <CheckBox checked={true}>I have read the manual</CheckBox>
        </Form.Item>
        <Form.Item wrapperTol={5}>
          <Button type="primary">Submit</Button>
        </Form.Item>
      </Form>
    </div>
  )
}
```

看到第一个 Input，用户需要自己控制变量的改变，这样的 Form 其实就是展示性的作用了，并没有实际意义，接下来就来到了受控表单。

# 受控表单

受控表单大致思路是这样的：Form 组件暴露给用户调用端一个方法集合，用户调用方法，如 onSubmit、resetFields、useFormContext 等可以提交、重置、校验表单，并且接受到 Form 处理完后的数据，比如哪些受控项校验失败、最终提交的结果是 true or false 等等，说白了就是把控制权完全的交给了 Form 组件，用户只需要在合适的时间做操作即可。

先看一段受控表单的代码：

```typescript
export default function index1() {
  const form = Form.useForm(); //使用Form组件回传的hooks，调用组件内链方法
  const formRef = createRef(); //调用端设一个ref，保证单页面多表单唯一性

  const submit = () => {
    const submitParams = form.onSubmit(formRef);
    console.log(submitParams);
  };

  return (
    <div>
      <Form layout={'horizontal'} formField={formRef} style={{ width: '600px' }}>
        <Form.Item
          label="Username"
          field="username"
          rules={[
            { required: true, message: '请输入用户名' },
            { maxLength: 10, message: '最大长度为10位' },
            { minLength: 3, message: '最小长度为3位' },
            { fn: (a: string) => a.includes('a'), message: '必须包含a' },
          ]}
        >
          <Input placeholder="Please enter your usename" width="200"></Input>
        </Form.Item>
        <Form.Item label="Post" field="post">
          <Input placeholder="Please enter your post" width="200"></Input>
        </Form.Item>
        <Form.Item label="Name" field="name" rules={[{ required: true, message: '请输入名字' }]}>
          <Select option={option} width={200} placeholder={'请选择'} />
        </Form.Item>
        <Form.Item
          label="CreateTime"
          field="CreateTime"
          rules={[{ required: true, message: '请输入名字' }]}
        >
          <TimePicker type="primary" showRange showClear />
        </Form.Item>
        <Form.Item wrapperTol={20}>
          <CheckBox checked={true}>I have read the manual</CheckBox>
        </Form.Item>
        <Form.Item wrapperTol={5}>
          <Button type="primary" handleClick={submit}>
            Submit
          </Button>
          <Button
            type="text"
            handleClick={() => form.resetFields(formRef)}
            style={{ margin: '0 10px' }}
          >
            Reset
          </Button>
        </Form.Item>
      </Form>
    </div>
  );
```

对比非受控表单有这些新增点：

1.  有些 Form.Item 多了 field 属性值，这就是给每一行做唯一标识的值，很重要；
2.  有些 Form.Item 多了 rules 属性值，这是用于校验的，很重要;
3.  使用 createRef 创建了一个 ref，并传给了 Form，这是让 Form 辨识指挥 Form 的是哪一个表单（以防单页面多表单混乱）;
4.  Form.useForm，用于调用 Form 提供的一系列方法，上文所说到，并且传参都是上一点的 ref;

而受控主要都是在 Form 中所做的。

先看一下 Form.tsx 中的关键代码：

```typescript
useEffect(() => {
  collectFormFns.onSubmit = onSubmit
  collectFormFns.resetFields = resetFields
  collectFormFns.validateFields = validateFields
  collectFormFns.useFormContext = useFormContext
  collectFormFns.formRef = formField
}, [fieldList])
```

这就是回传给用户 Form.useForm hook 的方法集合，可以用来调用，而这些方法是怎么写出来的呢？Form 在渲染的时候会根据 children 属性（所有 FormItem）中收集各自 prop 值，并且判断，如果有 field 则该 Item 需要被控制，收集该 Item 的 rules 等参数。

```typescript
const [fieldList, setFieldList] = useState<any>({});
...
useEffect(() => {
   const fieldL: any = {};
   children.forEach((child: any) => {
     if (child.props.field) {
       const key = child.props.field;
       fieldL[key] = {};
       fieldL[key].rules = child.props.rules || null;
     }
   });
   setFieldList(fieldL);
 }, []);
```

实现代码是这样的。

有了 fieldList，其实接下来就是编写 hook 中的函数，并且在用户调用函数时收集 FormItem 实时的参数，代码如下：

```typescript
const outputFormData = (ref: Ref<T> | null) => {
  //生成表体内容
  const returnField: any = {}
  let fieldType = ""
  for (var key in fieldList) {
    getDomVal((ref as any).current.querySelector(` .form-item .${key}`), key)
  }
  function getDomVal(dom: any, field: string) {
    if (dom?.childNodes.length === 0) {
      if (fieldType === "input") {
        returnField[field] = dom.value
      } else if (fieldType === "select") {
        if (dom.parentNode.getAttribute("class") === "placeholder") {
          returnField[field] = ""
        } else {
          returnField[field] = dom.parentNode.innerText
        }
      }
      fieldType = ""
    } else {
      if (dom !== null) {
        if (fieldType === "") {
          switch (dom.getAttribute("class")) {
            case "select":
              fieldType = "select"
              break
            case "box":
              fieldType = "input"
              break
          }
        }
        getDomVal(dom.childNodes[0], field)
      }
    }
  }
  return returnField
}
const onSubmit = (ref: Ref<T> | null) => {
  //表单提交
  const result = outputFormData(ref)
  const ruleResult = validateFields(result, ref)
  if (Object.keys(ruleResult).length > 0) {
    return { ...{ submitResult: false }, ruleResult }
  }
  return { ...{ submitResult: true }, result }
}

const validateFields = (resultField: any, ref: Ref<T> | null) => {
  //表单校验
  //表单校验
  if (resultField === undefined) {
    resultField = outputFormData(ref)
  }
  const resultRules: any = {}
  for (var key in resultField) {
    const field = fieldList[key]
    if (field.rules) {
      let isPass = true
      const rules = fieldList[key].rules
      rules.forEach((rule: ruleType) => {
        if (rule.required && resultField[key] == "" && isPass) {
          isPass = false
          changeValidateText(` .form-item .${key}`, rule.message, key, ref)
        } else if (
          rule.maxLength &&
          resultField[key].length > rule.maxLength &&
          isPass
        ) {
          isPass = false
          changeValidateText(` .form-item .${key}`, rule.message, key, ref)
        } else if (
          rule.minLength &&
          resultField[key].length < rule.minLength &&
          isPass
        ) {
          isPass = false
          changeValidateText(` .form-item .${key}`, rule.message, key, ref)
        } else {
          if (rule.fn && !rule.fn(resultField[key])) {
            isPass = false
            changeValidateText(` .form-item .${key}`, rule.message, key, ref)
          }
        }
        if (
          isPass &&
          (ref as any).current.querySelector(
            ` .form-item .${key} .show-rule-label`
          )
        ) {
          ;(ref as any).current
            .querySelector(` .form-item .${key} .show-rule-label`)
            ?.setAttribute("class", "hide-rule-label")
        }
      })
    }
  }
  function changeValidateText(
    className: string,
    text: string,
    field: string,
    ref: Ref<T | unknown> | null
  ) {
    resultRules[field] = text
    const hideDom = (ref as any).current.querySelector(
      `${className} .hide-rule-label`
    ) as HTMLElement
    const showDom = (ref as any).current.querySelector(
      `${className} .show-rule-label`
    ) as HTMLElement
    if (hideDom) {
      hideDom.innerText = text
    } else {
      showDom.innerText = text
    }
    hideDom?.setAttribute("class", "show-rule-label")
  }
  return resultRules
}
const resetFields = (ref: Ref<T | unknown> | null) => {
  //重置表单
  let fieldType = ""
  for (var key in fieldList) {
    getDomVal((ref as any).current.querySelector(` .form-item .${key}`), key)
  }
  function getDomVal(dom: any, field: string) {
    if (dom?.childNodes.length === 0) {
      if (fieldType === "input") {
        dom.value = ""
      } else if (
        fieldType === "select" &&
        (ref as any).current.querySelector(".size") !== null
      ) {
        ;((ref as any).current.querySelector(
          ".size"
        ) as HTMLElement).innerText = "请选择"
        ;(ref as any).current
          .querySelector(".size")
          ?.setAttribute("class", "placeholder")
      } else if (fieldType === "datePicker") {
        const datePickerInputs = (ref as any).current.querySelectorAll(
          ".rangePicker input"
        )
        datePickerInputs[0].value = getNowTime(false).split(" ")[0]
        if (datePickerInputs.length === 2) {
          const endDay: Array<string | number> = getNowTime(false)
            .split(" ")[0]
            .split("-")
          endDay[1] = (Number(endDay[1]) + 1) as number
          datePickerInputs[1].value = endDay.join("-")
        }
      }
      fieldType = ""
    } else {
      if (dom !== null) {
        if (fieldType === "") {
          switch (dom.getAttribute("class")) {
            case "select":
              fieldType = "select"
              break
            case "box":
              fieldType = "input"
              break
            case "rangePicker":
              fieldType = "datePicker"
          }
        }
        getDomVal(dom.childNodes[0], field)
      }
    }
  }
}
const useFormContext = (ref: Ref<T> | null) => {
  return outputFormData(ref)
}
```

就这样，受控表单完成了，整个表单的交互性也很强了，就像图中这样：

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/43e54b3a0c704994ad06be59f420a9a0~tplv-k3u1fbpfcp-zoom-1.image)

# 源码

Form.tsx:

```typescript
import React, { createContext, Ref, useEffect, useState } from "react"
import FormItem from "./form-item"
import { FormProps, ruleType } from "./interface"
import "./styles/index.module.less"
import { getNowTime } from "../_util/getNowTime"

export const ctx = createContext<any>({} as any) //顶层通信装置

export interface FormComponent {
  Item: typeof FormItem
}
export interface FromRefFunctions {
  formRef: string
  onSubmit: Function
  resetFields: Function
  validateFields: Function
  useFormContext: Function
}
export type fieldListType = {
  rules?: Array<any>
  field?: string
}
const collectFormFns: FromRefFunctions = {
  formRef: "",
  onSubmit: () => {},
  resetFields: () => {},
  validateFields: () => {},
  useFormContext: () => {},
}

const Form = <T>(props: FormProps<T>) => {
  const { children, layout = "horizontal", style, formField, disabled } = props

  const [fieldList, setFieldList] = useState<any>({})

  //根组件状态管理，向下传入
  const providerList = {
    layout,
  }

  const outputFormData = (ref: Ref<T> | null) => {
    //生成表体内容
    const returnField: any = {}
    let fieldType = ""
    for (var key in fieldList) {
      getDomVal((ref as any).current.querySelector(` .form-item .${key}`), key)
    }
    function getDomVal(dom: any, field: string) {
      if (dom?.childNodes.length === 0) {
        if (fieldType === "input") {
          returnField[field] = dom.value
        } else if (fieldType === "select") {
          if (dom.parentNode.getAttribute("class") === "placeholder") {
            returnField[field] = ""
          } else {
            returnField[field] = dom.parentNode.innerText
          }
        }
        fieldType = ""
      } else {
        if (dom !== null) {
          if (fieldType === "") {
            switch (dom.getAttribute("class")) {
              case "select":
                fieldType = "select"
                break
              case "box":
                fieldType = "input"
                break
            }
          }
          getDomVal(dom.childNodes[0], field)
        }
      }
    }
    return returnField
  }
  const onSubmit = (ref: Ref<T> | null) => {
    //表单提交
    const result = outputFormData(ref)
    const ruleResult = validateFields(result, ref)
    if (Object.keys(ruleResult).length > 0) {
      return { ...{ submitResult: false }, ruleResult }
    }
    return { ...{ submitResult: true }, result }
  }

  const validateFields = (resultField: any, ref: Ref<T> | null) => {
    //表单校验
    //表单校验
    if (resultField === undefined) {
      resultField = outputFormData(ref)
    }
    const resultRules: any = {}
    for (var key in resultField) {
      const field = fieldList[key]
      if (field.rules) {
        let isPass = true
        const rules = fieldList[key].rules
        rules.forEach((rule: ruleType) => {
          if (rule.required && resultField[key] == "" && isPass) {
            isPass = false
            changeValidateText(` .form-item .${key}`, rule.message, key, ref)
          } else if (
            rule.maxLength &&
            resultField[key].length > rule.maxLength &&
            isPass
          ) {
            isPass = false
            changeValidateText(` .form-item .${key}`, rule.message, key, ref)
          } else if (
            rule.minLength &&
            resultField[key].length < rule.minLength &&
            isPass
          ) {
            isPass = false
            changeValidateText(` .form-item .${key}`, rule.message, key, ref)
          } else {
            if (rule.fn && !rule.fn(resultField[key])) {
              isPass = false
              changeValidateText(` .form-item .${key}`, rule.message, key, ref)
            }
          }
          if (
            isPass &&
            (ref as any).current.querySelector(
              ` .form-item .${key} .show-rule-label`
            )
          ) {
            ;(ref as any).current
              .querySelector(` .form-item .${key} .show-rule-label`)
              ?.setAttribute("class", "hide-rule-label")
          }
        })
      }
    }
    function changeValidateText(
      className: string,
      text: string,
      field: string,
      ref: Ref<T | unknown> | null
    ) {
      resultRules[field] = text
      const hideDom = (ref as any).current.querySelector(
        `${className} .hide-rule-label`
      ) as HTMLElement
      const showDom = (ref as any).current.querySelector(
        `${className} .show-rule-label`
      ) as HTMLElement
      if (hideDom) {
        hideDom.innerText = text
      } else {
        showDom.innerText = text
      }
      hideDom?.setAttribute("class", "show-rule-label")
    }
    return resultRules
  }
  const resetFields = (ref: Ref<T | unknown> | null) => {
    //重置表单
    let fieldType = ""
    for (var key in fieldList) {
      getDomVal((ref as any).current.querySelector(` .form-item .${key}`), key)
    }
    function getDomVal(dom: any, field: string) {
      if (dom?.childNodes.length === 0) {
        if (fieldType === "input") {
          dom.value = ""
        } else if (
          fieldType === "select" &&
          (ref as any).current.querySelector(".size") !== null
        ) {
          ;((ref as any).current.querySelector(
            ".size"
          ) as HTMLElement).innerText = "请选择"
          ;(ref as any).current
            .querySelector(".size")
            ?.setAttribute("class", "placeholder")
        } else if (fieldType === "datePicker") {
          const datePickerInputs = (ref as any).current.querySelectorAll(
            ".rangePicker input"
          )
          datePickerInputs[0].value = getNowTime(false).split(" ")[0]
          if (datePickerInputs.length === 2) {
            const endDay: Array<string | number> = getNowTime(false)
              .split(" ")[0]
              .split("-")
            endDay[1] = (Number(endDay[1]) + 1) as number
            datePickerInputs[1].value = endDay.join("-")
          }
        }
        fieldType = ""
      } else {
        if (dom !== null) {
          if (fieldType === "") {
            switch (dom.getAttribute("class")) {
              case "select":
                fieldType = "select"
                break
              case "box":
                fieldType = "input"
                break
              case "rangePicker":
                fieldType = "datePicker"
            }
          }
          getDomVal(dom.childNodes[0], field)
        }
      }
    }
  }
  const useFormContext = (ref: Ref<T> | null) => {
    return outputFormData(ref)
  }
  useEffect(() => {
    const fieldL: any = {}
    children.forEach((child: any) => {
      if (child.props.field) {
        const key = child.props.field
        fieldL[key] = {}
        fieldL[key].rules = child.props.rules || null
      }
    })
    setFieldList(fieldL)
  }, [])
  useEffect(() => {
    collectFormFns.onSubmit = onSubmit
    collectFormFns.resetFields = resetFields
    collectFormFns.validateFields = validateFields
    collectFormFns.useFormContext = useFormContext
    collectFormFns.formRef = formField
  }, [fieldList])

  return (
    <ctx.Provider value={providerList}>
      <div className="form" style={style} ref={formField || null}>
        {disabled && <div className="disabled" />}
        {children}
      </div>
    </ctx.Provider>
  )
}

Form.Item = FormItem
Form.useForm = () => {
  return collectFormFns
}

export default Form
```

FormItem.tsx:

```typescript
import React, {
  useEffect,
  useState,
  useCallback,
  useContext,
  createRef,
} from "react"
import { FormItemProps } from "./interface"
import { FormItemAttrs, FormItemLabel } from "./classes"
import { ctx } from "./index"
import "./styles/form-item.module.less"

const FormItem = (props: FormItemProps) => {
  const {
    children,
    style = {},
    label,
    wrapperCol = 0,
    wrapperTol = 0,
    field,
    rules = [],
    disabled = false,
  } = props

  const [propsStyle, setPropsStyle] = useState({})
  const [labelStyle, setLabelStyle] = useState({})

  const Ctx = (function() {
    //创建一个ctx单例，防止组件内污染全局变量
    const c = useContext(ctx)
    return {
      get: (prop: string) => {
        return c[prop] || null
      },
    }
  })()

  useEffect(() => {
    setPropsStyle({ ...getPropsStyles(), ...style })
    setLabelStyle(getLabelPropsStyle())
  }, [props])

  const getPropsStyles = useCallback(() => {
    //基于props，动态构建一个props style集合
    const formAttrs = new FormItemAttrs(
      wrapperCol,
      wrapperTol,
      Ctx.get("layout")
    )
    return formAttrs.getStyle()
  }, [wrapperCol, wrapperTol, Ctx.get("layout")])
  const getLabelPropsStyle = useCallback(() => {
    //基于props，动态构建一个label props style集合
    const labelAttrs = new FormItemLabel(Ctx.get("layout"))
    return labelAttrs.getStyle()
  }, [Ctx.get("layout")])

  return (
    <div className="form-item" style={propsStyle}>
      <div className="label" style={labelStyle}>
        {rules.length > 0 && (
          <svg
            fill="currentColor"
            viewBox="0 0 1024 1024"
            width="0.5em"
            height="0.5em"
          >
            <path d="M583.338667 17.066667c18.773333 0 34.133333 15.36 34.133333 34.133333v349.013333l313.344-101.888a34.133333 34.133333 0 0 1 43.008 22.016l42.154667 129.706667a34.133333 34.133333 0 0 1-21.845334 43.178667l-315.733333 102.4 208.896 287.744a34.133333 34.133333 0 0 1-7.509333 47.786666l-110.421334 80.213334a34.133333 34.133333 0 0 1-47.786666-7.509334L505.685333 706.218667 288.426667 1005.226667a34.133333 34.133333 0 0 1-47.786667 7.509333l-110.421333-80.213333a34.133333 34.133333 0 0 1-7.509334-47.786667l214.186667-295.253333L29.013333 489.813333a34.133333 34.133333 0 0 1-22.016-43.008l42.154667-129.877333a34.133333 34.133333 0 0 1 43.008-22.016l320.512 104.106667L412.672 51.2c0-18.773333 15.36-34.133333 34.133333-34.133333h136.533334z"></path>
          </svg>
        )}
        {label || ""}
      </div>
      <div
        className={field || "content"}
        style={
          Ctx.get("layout") === "horizontal" ? { position: "relative" } : {}
        }
      >
        {children}
        {disabled && <div className="form-item-disabled"></div>}
        {field && rules.length > 0 && (
          <div className="hide-rule-label">{rules[0].message}</div>
        )}
      </div>
    </div>
  )
}

export default FormItem
```

具体的引入方法博主就不贴了，有想了解和使用的可以看下 github 哈~

# 组件测试

使用 jest 测试上述组件文档每个场景的业务正确性，具体测试代码 Form.test.tsx 如下：

```typescript
import React, { createRef } from "react"
import Form from "../../Form/index"
import Input from "../../Input"
import CheckBox from "../../CheckBox"
import Button from "../../Button"
import Select from "../../Select"
import Enzyme from "../setup"
import mountTest from "../mountTest"

const option = [
  {
    label: "Mucy",
    value: "mucy",
  },
  {
    label: "Mike",
    value: "mike",
  },
  {
    label: "aMck",
    value: "amck",
  },
]

const { mount } = Enzyme

mountTest(Form)

describe("Form", () => {
  it("test base form show correctly", () => {
    const component = mount(
      <Form layout={"vertical"} style={{ width: "600px" }}>
        <Form.Item label="Username">
          <Input placeholder="Please enter your usename" width="200"></Input>
        </Form.Item>
        <Form.Item label="Post">
          <Input placeholder="Please enter your post" width="200"></Input>
        </Form.Item>
        <Form.Item wrapperTol={20}>
          <CheckBox checked={true}>I have read the manual</CheckBox>
        </Form.Item>
        <Form.Item wrapperTol={5}>
          <Button type="primary">Submit</Button>
        </Form.Item>
      </Form>
    )
    const formDom = component.find(".form")
    expect(component.find(".form")).toHaveLength(1)
    expect(formDom.getDOMNode().getAttribute("style")).toEqual("width: 600px;")
    expect(component.find(".form .form-item")).toHaveLength(4)
    expect(
      component
        .find(".form .form-item")
        .at(2)
        .getDOMNode()
        .getAttribute("style")
        ?.includes("margin-bottom: 20px; margin-top: 40px;")
    ).toBe(true)
    expect(
      component
        .find(".form .form-item")
        .at(3)
        .getDOMNode()
        .getAttribute("style")
        ?.includes("margin-bottom: 20px; margin-top: 25px;")
    ).toBe(true)
  })
  it("test layout form show correctly", () => {
    const component = mount(
      <Form layout={"horizontal"} style={{ width: "600px" }}>
        <Form.Item label="Username">
          <Input placeholder="Please enter your usename" width="200"></Input>
        </Form.Item>
        <Form.Item label="Post">
          <Input placeholder="Please enter your post" width="200"></Input>
        </Form.Item>
        <Form.Item wrapperTol={20}>
          <CheckBox checked={true}>I have read the manual</CheckBox>
        </Form.Item>
        <Form.Item wrapperTol={5}>
          <Button type="primary">Submit</Button>
        </Form.Item>
      </Form>
    )
    expect(
      component
        .find(".form .form-item")
        .at(0)
        .getDOMNode()
        .getAttribute("style")
        ?.includes("flex-direction: column; align-items: flex-start;")
    ).toBe(false)
  })
  it("test control form use correctly", () => {
    const mockSubmitFn = jest.fn()
    const form = Form.useForm()
    const formRef = createRef()

    const component = mount(
      <Form layout={"vertical"} formField={formRef} style={{ width: "600px" }}>
        <Form.Item
          label="Username"
          field="username"
          rules={[
            { required: true, message: "请输入用户名" },
            { maxLength: 10, message: "最大长度为10位" },
            { minLength: 3, message: "最小长度为3位" },
          ]}
        >
          <Input placeholder="Please enter your usename" width="200"></Input>
        </Form.Item>
        <Form.Item label="Post" field="post">
          <Input placeholder="Please enter your post" width="200"></Input>
        </Form.Item>
        <Form.Item
          label="Name"
          field="name"
          rules={[{ required: true, message: "请输入名字" }]}
        >
          <Select option={option} width={200} placeholder={"请选择"} />
        </Form.Item>
        <Form.Item wrapperTol={20}>
          <CheckBox checked={true}>I have read the manual</CheckBox>
        </Form.Item>
        <Form.Item wrapperTol={5}>
          <Button type="primary" handleClick={mockSubmitFn}>
            Submit
          </Button>
        </Form.Item>
      </Form>
    )
    component
      .find(".form .form-item")
      .at(4)
      .find("button")
      .simulate("click")
    expect(mockSubmitFn).toBeCalled()
    const { submitResult, ruleResult } = form.onSubmit(formRef)
    const { name, username } = ruleResult
    expect(name).toEqual("请输入名字")
    expect(username).toEqual("请输入用户名")
    expect(submitResult).toEqual(false)
  })
  it("test form reset correctly", () => {
    const form = Form.useForm()
    const formRef = createRef()
    const mockResetFn = jest.fn()

    const component = mount(
      <Form
        layout={"horizontal"}
        formField={formRef}
        style={{ width: "600px" }}
      >
        <Form.Item
          label="Username"
          field="username"
          rules={[
            { required: true, message: "请输入用户名" },
            { maxLength: 10, message: "最大长度为10位" },
            { minLength: 3, message: "最小长度为3位" },
          ]}
        >
          <Input placeholder="Please enter your usename" width="200"></Input>
        </Form.Item>
        <Form.Item label="Post" field="post">
          <Input placeholder="Please enter your post" width="200"></Input>
        </Form.Item>
        <Form.Item
          label="Name"
          field="name"
          rules={[{ required: true, message: "请输入名字" }]}
        >
          <Select option={option} width={200} placeholder={"请选择"} />
        </Form.Item>
        <Form.Item wrapperTol={20}>
          <CheckBox checked={true}>I have read the manual</CheckBox>
        </Form.Item>
        <Form.Item wrapperTol={5}>
          <Button
            type="text"
            handleClick={mockResetFn}
            style={{ margin: "0 10px" }}
          >
            Reset
          </Button>
        </Form.Item>
      </Form>
    )
    component
      .find(".form .form-item")
      .at(0)
      .find("input")
      .simulate("change", {
        target: {
          value: "123",
        },
      })
    component
      .find(".form .form-item")
      .at(4)
      .find("button")
      .simulate("click")
    expect(mockResetFn).toBeCalled()
    form.resetFields(formRef)
    expect(
      component
        .find(".form .form-item")
        .at(0)
        .find("input")
        .getDOMNode()
        .getAttribute("value")
    )
  })

  it("test global disabled form show correctly", () => {
    const component = mount(
      <Form layout={"vertical"} disabled style={{ width: "600px" }}>
        <Form.Item
          label="Username"
          field="username"
          rules={[
            { required: true, message: "请输入用户名" },
            { maxLength: 10, message: "最大长度为10位" },
            { minLength: 3, message: "最小长度为3位" },
          ]}
        >
          <Input placeholder="Please enter your usename" width="200"></Input>
        </Form.Item>
        <Form.Item label="Post" field="post">
          <Input placeholder="Please enter your post" width="200"></Input>
        </Form.Item>
        <Form.Item
          label="Name"
          field="name"
          rules={[{ required: true, message: "请输入名字" }]}
        >
          <Select option={option} width={200} placeholder={"请选择"} />
        </Form.Item>
        <Form.Item wrapperTol={20}>
          <CheckBox checked={true}>I have read the manual</CheckBox>
        </Form.Item>
        <Form.Item wrapperTol={5}>
          <Button type="primary">Submit</Button>
        </Form.Item>
      </Form>
    )
    expect(component.find(".form .disabled")).toHaveLength(1)
  })

  it("test Form.Item disabled form show correctly", () => {
    const component = mount(
      <Form layout={"vertical"} style={{ width: "600px" }}>
        <Form.Item
          label="Username"
          field="username"
          rules={[
            { required: true, message: "请输入用户名" },
            { maxLength: 10, message: "最大长度为10位" },
            { minLength: 3, message: "最小长度为3位" },
            { fn: (a: string) => a.includes("a"), message: "必须包含a" },
          ]}
        >
          <Input placeholder="Please enter your usename" width="200"></Input>
        </Form.Item>
        <Form.Item label="Post" field="post" disabled>
          <Input placeholder="Please enter your post" width="200"></Input>
        </Form.Item>
        <Form.Item
          label="Name"
          field="name"
          rules={[{ required: true, message: "请输入名字" }]}
        >
          <Select option={option} width={200} placeholder={"请选择"} />
        </Form.Item>
        <Form.Item wrapperTol={20}>
          <CheckBox checked={true}>I have read the manual</CheckBox>
        </Form.Item>
        <Form.Item wrapperTol={5}>
          <Button type="primary">Submit</Button>
        </Form.Item>
      </Form>
    )
    expect(
      component
        .find(".form .form-item")
        .at(1)
        .find(".form-item-disabled")
    ).toHaveLength(1)
  })

  it("test useFormContext api correctly", () => {
    const form = Form.useForm()
    const formRef = createRef()
    const mockFn = jest.fn()

    const component = mount(
      <Form layout={"vertical"} formField={formRef} style={{ width: "600px" }}>
        <Form.Item
          field="username"
          rules={[
            { required: true, message: "请输入用户名" },
            { maxLength: 10, message: "最大长度为10位" },
            { minLength: 3, message: "最小长度为3位" },
            { fn: (a: string) => a.includes("a"), message: "必须包含a" },
          ]}
        >
          <Input placeholder="Please enter your usename" width="240"></Input>
        </Form.Item>
        <Form.Item
          field="phone"
          rules={[{ required: true, message: "请输入手机号" }]}
        >
          <Input
            placeholder="Please enter your phone number"
            width="240"
          ></Input>
        </Form.Item>
        <Form.Item wrapperTol={5}>
          <Button type="primary" handleClick={mockFn}>
            Register
          </Button>
        </Form.Item>
      </Form>
    )
    component
      .find(".form .form-item")
      .at(0)
      .find("input")
      .simulate("change", {
        target: {
          value: "123",
        },
      })
    component
      .find(".form .form-item")
      .at(1)
      .find("input")
      .simulate("change", {
        target: {
          value: "123",
        },
      })
    component
      .find(".form .form-item")
      .at(2)
      .find("button")
      .simulate("click")
    expect(mockFn).toBeCalled()
    const { username, phone } = form.useFormContext(formRef)
    expect(username).toEqual("123")
    expect(phone).toEqual("123")
  })
})
```

# 组件库地址

- Concis 组件库线上链接：[http://react-view-ui.com:92](http://react-view-ui.com:92)
- github：[https://github.com/fengxinhhh/Concis](https://github.com/fengxinhhh/Concis)
- npm：[https://www.npmjs.com/package/concis](https://www.npmjs.com/package/concis)

开源不易，欢迎学习和体验，喜欢请多多支持，有问题请留言，谢谢支持
