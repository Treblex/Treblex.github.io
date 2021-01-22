---
title: Go Web Server上传文件实践
intro: ""
featured_image: ""
date: 2020-04-21 15:05:28
---

<!-- 文件地址 -->

[https://github.com/Treblex/go-echo-demo/blob/master/server/router/router.go#L63](https://github.com/Treblex/go-echo-demo/blob/master/server/router/router.go#L63)

```go
func upload(c echo.Context) error {
	file, err := c.FormFile("file")
	if err != nil {
		return util.JSONErr(c, err, "上传错误") //未获取到文件流
	}
	pathExt := path.Ext(file.Filename)
	acceptsImgExt := []interface{}{"jpg", "png", "jpeg", "webp"}           //图片类型
	acceptsVideoExt := []interface{}{"mov", "mp4", "avi"}                  //视频类型
	acceptsOtherFileExt := []interface{}{"pdf", "zip", "rar", "gz", "txt"} //其他文件类型
	folder := ""
	// 如果符合类型，设定目录
	if inArray(acceptsImgExt, strings.Trim(pathExt, ".")) {
		folder = "image"
	}
	if inArray(acceptsVideoExt, strings.Trim(pathExt, ".")) {
		folder = "video"
	}
	if inArray(acceptsOtherFileExt, strings.Trim(pathExt, ".")) {
		folder = "file"
	}
	// 如果不符合任何一种类型
	if folder == "" {
		return util.JSONErr(c, nil, "文件不合法")
	}

	// 打开文件流
	src, err := file.Open()
	if err != nil {
		return util.JSONErr(c, err, "打开文件失败")
	}
	defer src.Close() //函数结束时自动关闭文件

	//创建文件夹
	dir, err := getDir("./static/upload/"+folder+"/", time.Now().Format("2006_01_02"))
	if err != nil {
		return util.JSONErr(c, err, "创建文件夹失败")
	}

	// 随机文件名 + 文件后缀
	randName := util.RandStringBytes(32) + pathExt
	// Destination
	fileName := filepath.Join(dir, randName)

	// 创建空文件
	dst, err := os.Create(fileName)
	if err != nil {
		return util.JSONErr(c, err, "创建文件失败")
	}
	defer dst.Close()
	// Copy文件流到新建到文件
	if _, err = io.Copy(dst, src); err != nil {
		return util.JSONErr(c, err, "拷贝文件至目标失败")
	}
	// 拼接文件地址，不带协议头，方便处理http 到https升级 ， 其实也没找到协议头在哪儿，req对象里没有返回到空字符串
	return util.JSON(c, fmt.Sprintf("//%s/%s", c.Request().Host, fileName), "上传成功", 200)
}

// 创建文件夹
func getDir(path string, foderName string) (dir string, err error) {
	folder := filepath.Join(path, foderName)
	if _, err = os.Stat(folder); os.IsNotExist(err) {
		err = os.MkdirAll(folder, os.ModePerm)
		if err != nil {
			return
		}
	}
	dir = folder
	return
}

// 在数组中
func inArray(arr []interface{}, item interface{}) (inArr bool) {
	index := -1
	for i, x := range arr {
		if item == x {
			index = i
		}
	}
	return index > -1
}

```
