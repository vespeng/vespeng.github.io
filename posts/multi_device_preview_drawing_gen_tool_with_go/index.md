# 利用周末写一个小工具：多设备预览图生成


起因是这样的，每次在调整完自己网站的时候，对于一些 UI 样式的调整，都需要提交代码并构建好后，通过第三方的预览图生成网站或者手动修图来制作一个网站预览图并重新上传提交代码。
这样似乎有些繁琐了，尝试寻求一个完美的工具来达到这个目的，但在 github 上寻一圈未果，所以就利用这个周末去写一个简单的工具来实现这个需求。
<!--more-->

## 效果图

先来预览下效果图：

![预览图](./images/preview.png)

## 需求分析

1. 收集多设备的外壳图片素材，必须带有透明通道，方便后续图片的合并
2. 获取设备图片的屏幕尺寸（宽·高），截图的大小需贴合设备图片的尺寸
3. 获取网页截图，需根据设备 UA 进行读取网页（这里使用 `chromedp`），然后裁剪成设备屏幕的尺寸
4. 创建一个大的画布，将截图贴到画布上，接着将设备图片贴到截图上（这里需要考虑设备最终在画布上的坐标位置）
5. 考虑 IPhone 设备的四角非直角，所以需要特殊处理下，将设备图片的圆角部分进行裁剪

## 实现思路

- 素材获取： [https://mockuphone.com/type/all/](https://mockuphone.com/type/all/)
- 图片裁剪： [https://www.zaixianps.cc/ps/](https://www.zaixianps.cc/ps/)
- 图片大小调整： [https://www.iloveimg.com/zh-cn/resize-image](https://www.iloveimg.com/zh-cn/resize-image)

### Step 1

利用 `chromedp` 包，读取本机带有 Chromium 内核的浏览器，初始化浏览器分配上下文，采用无头模式

_ps：这里仅展示核心逻辑，完整代码请下滑到最底移步到 github 仓库查看_

```go {data-open=true}
// 初始化浏览器分配器上下文
browserPath, err := "C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe"
opts := append(chromedp.DefaultExecAllocatorOptions[:],
    chromedp.ExecPath(browserPath),
    chromedp.NoFirstRun,
    chromedp.NoDefaultBrowserCheck,
    chromedp.Headless,
)
allocCtx, cancel := chromedp.NewExecAllocator(context.Background(), opts...)
defer cancel()
```

并发读取网页，获取网页截图

```go {data-open=true}
func takeScreenshotForDevice(ctx context.Context, url string, width, height int) (*image.RGBA, error) {
	var buf []byte
	
    err := chromedp.Run(ctx,
        chromedp.EmulateViewport(int64(width), int64(height)),
        chromedp.Navigate(url),
        chromedp.Sleep(3*time.Second),
        chromedp.WaitVisible("body", chromedp.ByQuery),
        chromedp.CaptureScreenshot(&buf),
    )
    if err != nil {
        return nil, err
    }

	img, _, err := image.Decode(bytes.NewReader(buf))
	if err != nil {
		return nil, err
	}

	bounds := img.Bounds()
	rgba := image.NewRGBA(bounds)
	draw.Draw(rgba, bounds, img, bounds.Min, draw.Src)

	return rgba, nil
}
```

### Step 2

创建画布

```go
canvas := imaging.New(2560, 1600, color.White)
```

### Step 3

遍历所有截图并贴入画布，同时将设备外壳覆盖到截图上

_ps：这里逻辑取值较复杂，故省略代码_

```go {data-open=true}
for _, dev := range Devices {
	...
    // 读取设备图片
    ...

    // 解码图片数据
    ...

    // 转换为 RGBA 格式以便绘制
    ...

    // 将外壳覆盖到画布的对应位置（LayoutX/Y）
    ...
}
```

### Step 4

保存并输出

```go {data-open=true}
outFile := "output/preview.png"

f, err := os.Create(outFile)
if err != nil {
    panic(err)
}
defer f.Close()

if err := png.Encode(f, canvas); err != nil {
    panic(err)
}

fmt.Println("预览图生成成功:", outFile)
```

## 总结

**复杂点**：

- 设备在画布中的位置计算，以及截图的裁剪位置计算
- 设备图片的圆角处理，这里用 ai 给的代码做了处理，也算是被 ai 坑了一回，四角虽做了处理，但是并没有透明化，也是耗费了很长时间才弄懂图片透明逻辑的处理

**不足点**：

- 访问页面未设置超时，对于一些网站访问会比较慢，可能会导致卡死，不过访问本地页面处理速度还是很快的，这里后续有时间优化下
- 截图有时候内容会加载不全，问题同上

**最后**：

源码已上传至 github 仓库：[GitHub - vespeng/multi-device-preview](https://github.com/vespeng/multi-device-preview/)，欢迎 fork 和 star。


---

> 作者: [Vespeng](https://github.com/vespeng/)  
> URL: https://vespeng.com/posts/multi_device_preview_drawing_gen_tool_with_go/  

