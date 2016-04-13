+++
title  = "UIImagePickerControllerUsage"
isCJKLanguage = true
date = "2015-07-12T16:27:05"
draft = true
+++


周末两天完全没闲着，研究iOS研究的头都要裂了。

这次先说说UIImagePickerController这个类。

首先，它的作用是拍摄，既可以拍照片也可以录像。

基本流程：

1. 让你的类实现两个协议(`protocol`)
    - `UINavigationControllerDelegate`
    - `UIImagePickerControllerDelegate`
2. 在storyboard中添加一个按钮，并给这个按钮绑上一个`IBAction`，就叫做`takePictures`吧。
3. 实现这个`takePictures`

```
    // 判断当前的设备是否支持UIImagePickerControllerSourceTypeCamera，也就是是不是有摄像头，目前在网上能看到的代码里这一点都会做个判断，我觉得较早之前为iPod写程序并且要兼容iPhone时有需要，现在可能就已经不需要了，还有谁的iPhone没有摄像头啊
    if ([UIImagePickerController isSourceTypeAvailable:UIImagePickerControllerSourceTypeCamera]) {
    // 指定委托代理
        videoPicker.delegate = self;
    // 指定使用摄像头，因为sourceType还可能有UIImagePickerControllerSourceTypePhotoLibrary和UIImagePickerControllerSourceTypeSavedPhotosAlbum两种，而这两种是从手机上保存的照片中选择照片
        videoPicker.sourceType = UIImagePickerControllerSourceTypeCamera;
    // 这个是指定媒体的类型，也就是静态图片、视频、甚至无声视频这些
    // 在XCode的提示里有两个类型可能会引起混淆，kUTTypeMovie和kUTTypeVideo，你不看文档一定想不到两个的区别——前者是有声视频，后者则只有图像，没有声音
        videoPicker.mediaTypes = [NSArray arrayWithObjects:(NSString *)kUTTypeMovie, nil];
        // 这一步很关键，执行了这个拍照的界面才会在手机上呈现出来
        [self presentViewController:videoPicker animated:YES completion:nil];
    }
```

4. 保存拍摄的照片或视频
    这个依赖于前面实现的协议`UIImagePickerControllerDelegate`，该协议有一个`imagePickerController:didFinishPickingMediaWithInfo`方法，我们要做的保存动作就在这个方法里实现。
    
```
- (void)imagePickerController:(nonnull UIImagePickerController *)picker didFinishPickingMediaWithInfo:(nonnull NSDictionary<NSString *,id> *)info {
    // 获取拍摄的媒体的类型
    NSString *mediaType = [info objectForKey:UIImagePickerControllerMediaType];
    // 判断拍摄的媒体的类型
    if ([mediaType isEqualToString:(NSString *) kUTTypeMovie]) {
    // 获取要保存的文件的路径
        NSURL *movieURL = [info objectForKey:UIImagePickerControllerMediaURL];
    // 保存，后面的三个参数是用来处理异常的，这里只是一个demo，暂时忽略它们
        UISaveVideoAtPathToSavedPhotosAlbum([movieURL relativePath], nil, nil, nil);
        
    }
}
```
完成了上面的方法之后，拍摄的视频就会保存在Camera Roll里面了。但是这个项目还不完整，因为点了use，也就是保存了拍摄的东西之后App的还是停留在取景器界面，具体细节再慢慢研究。


