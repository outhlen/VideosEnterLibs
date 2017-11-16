# ShVideoDemo
Android 视频录制Demo 防微信小视频 视频压缩（FFmpeg）

 Gradle:
~~~gradle
compile 'com.github.outhlen:VideosEnterLibs:1.0.1'
~~~

  用法：
  
  1. 录制短视频 
  
                      //显示视频录制控件
      VideoInputDialog.show(getSupportFragmentManager(),XXX.this,VideoInputDialog.Q480,XXX.this)；
	  
	  回调函数：
	  
	   /**
     * 小视屏录制回调
     * @param path
     */
    @Override
    public void videoPathCall(String path) {

        Log.e("地址:",path);
        //根据视频地址获取缩略图
        this.path =path;
        Bitmap bitmap = ThumbnailUtils.createVideoThumbnail(path, MediaStore.Video.Thumbnails.MINI_KIND);
        image.setImageBitmap(bitmap);
        first.setText(getFileSize(path));

    }

   
	  
  2. 录制长视频文件
  
      VideoInputActivity.startActivityForResult(MainActivity.this, REQUEST_CODE_FOR_RECORD_VIDEO,VideoInputActivity.Q720);
	  
	  回调函数： 
	  
	  
	   /**
     * 录制视频回调
     * @param requestCode
     * @param resultCode
     * @param data
     */
    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        if(requestCode==REQUEST_CODE_FOR_RECORD_VIDEO&&resultCode==RESULT_CANCELED){

        }
        if(requestCode==REQUEST_CODE_FOR_RECORD_VIDEO&&resultCode==RESULT_OK){
            String path = data.getStringExtra(VideoInputActivity.INTENT_EXTRA_VIDEO_PATH);
            Log.e("地址:",path);
            //根据视频地址获取缩略图
            this.path =path;
            Bitmap bitmap = ThumbnailUtils.createVideoThumbnail(path, MediaStore.Video.Thumbnails.MINI_KIND);
            image.setImageBitmap(bitmap);
            first.setText(getFileSize(path));
        }
        super.onActivityResult(requestCode, resultCode, data);
    }

	3. 视频压缩处理：
	
	          //获取视频时长  计算压缩进度用
                MediaMetadataRetriever retr = new MediaMetadataRetriever();
                retr.setDataSource(path);
                String time = retr.extractMetadata(MediaMetadataRetriever.METADATA_KEY_DURATION);//获取视频时长
                //7680
                try {
                    videoLength = Double.parseDouble(time)/1000.00;
                } catch (Exception e) {
                    e.printStackTrace();
                    videoLength = 0.00;
                }
                Log.v(TAG, "videoLength = "+videoLength + "s");


                /**
                 * 压缩视频
                 */
                CompressorUtils compressorUtils = new CompressorUtils(path,currentOutputVideoPath,MainActivity.this);
                compressorUtils.execCommand(new CompressListener() {
                    @Override
                    public void onExecSuccess(String message) {
                        Log.i(TAG, "success " + message);
                        progressBar.setVisibility(View.INVISIBLE);
                        textAppend(getString(R.string.compress_succeed));
                        back.setText(getFileSize(currentOutputVideoPath));
                        //获取缩略图
                        Bitmap bitmap = ThumbnailUtils.createVideoThumbnail(currentOutputVideoPath, MediaStore.Video.Thumbnails.MINI_KIND);
                        imag2.setImageBitmap(bitmap);
                    }

                    @Override
                    public void onExecFail(String reason) {
                        Log.i(TAG, "fail " + reason);
                    }

                    @Override
                    public void onExecProgress(String message) {
                        progressBar.setVisibility(View.VISIBLE);
                        textAppend(getString(R.string.compress_progress, message));

                        int i = getProgress(message);
                        Log.e("进度",i+"");
                        progressBar.setProgress(i);
                    }
                });
            }
	
	/**
     * 录制视频回调
     * @param requestCode
     * @param resultCode
     * @param data
     */
    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        if(requestCode==REQUEST_CODE_FOR_RECORD_VIDEO&&resultCode==RESULT_CANCELED){

        }
        if(requestCode==REQUEST_CODE_FOR_RECORD_VIDEO&&resultCode==RESULT_OK){
            String path = data.getStringExtra(VideoInputActivity.INTENT_EXTRA_VIDEO_PATH);
            Log.e("地址:",path);
            //根据视频地址获取缩略图
            this.path =path;
            Bitmap bitmap = ThumbnailUtils.createVideoThumbnail(path, MediaStore.Video.Thumbnails.MINI_KIND);
            image.setImageBitmap(bitmap);
            first.setText(getFileSize(path));
        }
        super.onActivityResult(requestCode, resultCode, data);
    }

    public void openView(String path){
        if(TextUtils.isEmpty(path)){

            return;
        }
        File file = new File(path);
        SystemAppUtils.openFile(file,this);
    }



    private String getFileSize(String path) {
        File f = new File(path);
        if (!f.exists()) {
            return "0 MB";
        } else {
            long size = f.length();
            return (size / 1024f) / 1024f + "MB";
        }
    }
    int progress=0;
    private int getProgress(String source){
        // Duration: 00:00:22.50, start: 0.000000, bitrate: 13995 kb/s

        //progress frame=   28 fps=0.0 q=24.0 size= 107kB time=00:00:00.91 bitrate= 956.4kbits/s
        if(source.contains("start: 0.000000")){
            return progress;
        }
        Pattern p = Pattern.compile("00:\\d{2}:\\d{2}");
        Matcher m = p.matcher(source);
        if (m.find()) {
            //00:00:00
            String result = m.group(0);
            String temp[] = result.split(":");
            Double seconds = Double.parseDouble(temp[1]) * 60 + Double.parseDouble(temp[2]);

            if (0 != videoLength) {
                Log.v("进度长度", "current second = " + seconds+"/videoLength="+videoLength);
                progress = (int)(seconds *100/ videoLength);

                return progress;
            }
            return progress;
        }
        return progress;
    }


    private void textAppend(String text) {
        if (!TextUtils.isEmpty(text)) {
          Log.e("日志",text);
        }
    }
	
<font color=#ff0022ff size=7 face="黑体">注意:targetSdkVersion 23 及以上 要注意 6.0运行时权限 或干脆用23以下</font>

![](1.gif)