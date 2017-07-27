---
title:  "Android调起照相机、相册获取图片，并压缩图片"
date: 2017-07-25 14:47:01
categories: [Android,UI]
tags: [Android]
---


>最近封装了图片OCR模块，模块代码不方便贴出来，就简单记录一下图片的处理方式
获取图片方式是让用户选择 相机拍照 或者从 相册 中选择

![](http://ot0nm27pk.bkt.clouddn.com/gududemao.png)


先说照相机方式：

首先考虑Android6.0之后的动态权限处理
```java
  /**
     * 核对相机权限
     */
    private void checkCameraPermission() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            try {
                int isPermission = checkSelfPermission(Manifest.permission.CAMERA);
                if( isPermission == PackageManager.PERMISSION_GRANTED) {
                    openCarema();
                    return;
                }
                boolean shouldRequest = shouldShowRequestPermissionRationale(Manifest.permission.CAMERA);
                if (shouldRequest) {
                    requestCameraPermission();
                    return;
                }
                showToast("请打开摄像机权限");
            } catch (RuntimeException e) { //防止MUNU系统等，更改权限的名称
                showToast("请打开摄像机权限");
            }
        } else {
            openCarema();
        }
    }
      /**
     * 请求打开权限
     */
    @TargetApi(Build.VERSION_CODES.M)
    private void requestCameraPermission() {
        requestPermissions(new String[]{Manifest.permission.CAMERA}, REQUESTCODE_OPEN_CAMERA_PERMISSION);
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        if (requestCode == REQUESTCODE_OPEN_CAMERA_PERMISSION) {
            if (grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                openCarema();
            } else {
                showToast("请打开摄像机权限");
            }
        }
    }

```
上边代码很简单明了，这里就不做赘述了

下边打开相机
```java
 	/**
     * 打开相机
     */
    private void openCarema() {
        // 指定相机拍摄照片保存地址
        String state = Environment.getExternalStorageState();
        if (state.equals(Environment.MEDIA_MOUNTED)) {
            cameraPath = SAVED_IMAGE_DIR_PATH + System.currentTimeMillis() + ".png";
            Intent intent = new Intent();
            // 指定开启系统相机的Action
            intent.setAction(MediaStore.ACTION_IMAGE_CAPTURE);
            String out_file_path = SAVED_IMAGE_DIR_PATH;
            File dir = new File(out_file_path);
            if (!dir.exists()) {
                dir.mkdirs();
            } // 把文件地址转换成Uri格式
            Uri uri = Uri.fromFile(new File(cameraPath));
            // 设置系统相机拍摄照片完成后图片文件的存放地址
            intent.putExtra(MediaStore.EXTRA_OUTPUT, uri);
            startActivityForResult(intent, CAMERA_REQUEST_CODE);
        } else {
            showToast("请确认已经插入SD卡");
        }
    }
```
上面可以看到，开启系统现有相机应用拍摄照片，需要用的MediaStore.ACTION_IMAGE_CAPTURE作为Intent的action开启Activity即可。但是在使用系统现有相机用用的时候，默认会把图片保存到系统图库的目录下，如果需要指定图片文件的保存路径，需要额外在Intent中设置。

设置系统现有相机应用的拍摄照片的保存路径，需要用Intent.putExtra()方法通过MediaStore.EXTRA_OUTPUT去设置Intent的额外数据，这里传递的是一个Uri参数，可以是一个文件路径的Uri。

<b>如果得到结果的话，我们在讲完打开相册后再说</b>

相册方式
```java
//打开相册
Intent intent = new Intent(Intent.ACTION_PICK, null);
intent.setDataAndType(MediaStore.Images.Media.EXTERNAL_CONTENT_URI, "image/*");
intent.setAction(Intent.ACTION_GET_CONTENT);
startActivityForResult(intent, ALBUM_REQUEST_CODE);
```

处理返回结果
```java
  @Override
    public void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        Bitmap bitmap = null;
        if (resultCode == Activity.RESULT_OK) {
            switch (requestCode) {
                case CAMERA_REQUEST_CODE:
                    if (TextUtils.isEmpty(cameraPath)) {
                        showToast("获取摄像机图片失败");
                        return;
                    }
                    bitmap = BitmapUtils.getSmallBitmap(cameraPath, 480, 640);
                    Logger.d("path=" + cameraPath);
                    File file = new File(cameraPath);
                    if (file.exists()) {
                        file.delete();
                        Logger.d("删除相机图片");
                    }
                    break;
                case ALBUM_REQUEST_CODE:
                    Uri uri = data.getData();
                    final String absolutePath= getAbsolutePath(this, uri);
                    if (TextUtils.isEmpty(absolutePath)) {
                        showToast("获取摄像机图片失败");
                        return;
                    }
                    bitmap = BitmapUtils.getSmallBitmap(absolutePath, 480, 640);
                    break;
                default:
                    break;
            }
            if (bitmap == null) {
                showToast("获取图片失败");
                return;
            }
            fillBitmap(bitmap);
        }
    }
```

由上边可知BitmapUtils.getSmallBitma是得到压缩之后的图片，下边放代码
```java
    /**
     * 根据路径获得突破并压缩返回bitmap用于显示
     * 作者：QianqianLis
     * 链接：http://www.jianshu.com/p/81e553fd0bc3
     * 來源：简书
     * @param filePath
     * @return
     */
    public static Bitmap getSmallBitmap(String filePath, int reqWidth, int reqHeight) {
        final BitmapFactory.Options options = new BitmapFactory.Options();
        options.inJustDecodeBounds = true;  //只返回图片的大小信息
        BitmapFactory.decodeFile(filePath, options);
        // Calculate inSampleSize
        options.inSampleSize = calculateInSampleSize(options, reqWidth, reqHeight);
        // Decode bitmap with inSampleSize set
        options.inJustDecodeBounds = false;
        return BitmapFactory.decodeFile(filePath, options);
    }
```
这是从简书上看到的方法，只是做了一个简单的封装就使用了，亲自分别保存了压缩前的的图片和压缩后的图片，做了一下对比，红米note4上原图3M压缩完800k，当然这只是大概数据

