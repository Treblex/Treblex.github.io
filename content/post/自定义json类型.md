---
title: Go语言 gorm 自定义json类型
intro: ""
featured_image: ""
date: 2020-03-27 11:20:34
---

# 在 gorm 的 stuct 中实现自定义类型，

### 完成，

> req.body=>(MarshaJson json)=>stuct=>(Value grom.Model)=>数据库

> 数据库=>(Scan grom.Model)=>struct=>(UnmarshaJson json)=>json

```go


// Array json传数组类型 >>>修复：
type Array []string
//gorm中声明数据模型的时候需要 Images      util.Array `gorm:"type:MEDIUMTEXT" json:"images" `

// UnmarshalJSON req.body []byte=>对象，记得调用json.Unmarshal要新建原始类型进行绑定，不如会死循环
func (a *Array) UnmarshalJSON(b []byte) error {
	// b = bytes.Trim(b, "\"")
	// fmt.Printf("%v", string(b))

//这里一定要新建对象
//json.Unmarshal调用的是 stuct.UnmarshalJSON
//所以这里如果直接绑定 &a 会出现死循环
	arr := []string{}
	if err := json.Unmarshal(b, &arr); err != nil {
		return err //如果解码失败
	}
	*a = Array(arr) //解码成功赋值
	// fmt.Printf("Array UnmarshalJSON %v \n", arr)
	return nil
}

// Value 存库,对象到转储数据 标准字符 int类型
func (a Array) Value() (driver.Value, error) {
	// fmt.Printf("value %v \n", a)
	if len(a) == 0 {
		return nil, nil
    }
    //fmt.Sprint(a) []string=>[a,b,c] 强制转换为字符类型，类似与js中的 obj => [object,object]
    //strings.Trim  [a b c]=>a b c
    //strings.ReplaceAll a b c => a,b,c
	arr := strings.ReplaceAll(strings.Trim(fmt.Sprint(a), "[]"), " ", ",")
	return arr, nil
}

// Scan 绑定，数据库到对象,这里到数据取到到都是[]uint8字节，转化为对象
func (a *Array) Scan(v interface{}) error {
	value, ok := v.([]uint8) //自定义的类型 从数据库取出，scan的时候 varchar(255)和double 获取到的都是 []uint8 字节，可能哪里还有问题，之前看的一个自定义时间格式化的可以获取到time.Time类型
	if ok {
        // 转化字节到字符串 分割字符串
		arr := strings.Split(string(value), ",")
		*a = Array(arr)//重新赋值Arr
		return nil
	}
	return fmt.Errorf("%v 类型错误  scan失败", reflect.TypeOf(v))
}

// MarshalJSON 对象到json转换 接口展示
func (a *Array) MarshalJSON() ([]byte, error) {
    //这里在scan后边执行 直接解码，注意新建通用类型
	b, err := json.Marshal([]string(*a))
	// fmt.Printf("MarshalJSON %v \n", string(b))
	if err != nil {
		return nil, err
	}
	// 空数组默认值，空数组返回null在前端还听难受到其实
	if string(b) == "null" {
		b = []byte(`[]`)
	}
	return b, nil
}


```

## 实现效果

### 请求

```
    post("",{data:{arr:["asd","asd"]}})
```

#### 存库

```
   "asd,asd"
```

#### Response

```
    arr:["asd","asd"]
```
