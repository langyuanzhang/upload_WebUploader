# upload_WebUploader
上传文件和图片 （使用百度WebUploader插件）


```js
<!-- 上传 -->
<link rel="stylesheet" href="__AdminLTE__/bower_components/bootstrap/dist/css/bootstrap.min.css">
<link rel="stylesheet" href="__common__/css/webuploader.css">
```

```html
  <fieldset class="layui-elem-field layui-field-title" style="margin-top: 20px;">
    <legend>上传附件</legend>
  </fieldset>
  <div class="layui-form-item" hidden>
    <label class="layui-form-label">附件id</label>
    <div class="layui-input-block">
      <input type="text" name="file_id" id="file_id" class="layui-input" readonly="readonly">
    </div>
  </div>

  <div id="uploader" class="wu-example">
    <div id="list"> </div>
    <!--用来存放文件信息-->
    <div id="thelist" class="uploader-list"></div>
    <div style="display: flex; flex-direction: row;">
      <div id="picker" style="margin-right: 10px">选择文件</div>
      <div id="ctlBtn"  style="background: red;width: 80px;text-align: center;line-height: 37px; height: 37px;color: #fff;border-radius: 3px;cursor: pointer;">开始上传</div>
    </div>
  </div>
```

```js
<!-- 上传 -->
<script type="text/javascript" src="__common__/js/webuploader.min.js"></script>

<script type="text/javascript">
  $(function () {
    var options = {
      type: 'post', //post提交
      //url:'', 
      dataType: "json", //json格式
      // data:{id:111},    //如果需要提交附加参数，视情况添加
      clearForm: false, //成功提交后，清除所有表单元素的值
      resetForm: false, //成功提交后，重置所有表单元素的值
      cache: false,
      async: false, //同步返回
      success: function (data) {
        if (data.code == 0) {
          layer.msg(data.msg);
        } else {
          layer.msg(data.msg, {
            icon: 1,
            time: 500
          }, function () {
            $("#reset").click();
            lotus_close();
            parent.location.reload();
          });
        }
        //服务器端返回处理逻辑
      },
      error: function (XmlHttpRequest, textStatus, errorThrown) {
        layer.msg('操作失败:服务器处理失败');
      }
    };
    layui.use('form', function () {
      var form = layui.form;
      $('#mainForm').ajaxForm(options).submit(function (data) {});
    });

  })
  // });
</script>


<!-- 引入js 使用上传功能-->
<script type="text/javascript">
  $(function () {
    //定义附件id
    var allfile='';

    var uploader = WebUploader.create({
      resize: false, // 不压缩image      
      swf: '__common__/js/Uploader.swf', // swf文件路径
      server: "{:url('admin/contract/uplodas')}", // 文件接收服务端。
      pick: '#picker', // 选择文件的按钮。可选
      chunked: true,//开启分片上传
      // chunkSize:3*1024*1024,
      // threads: 1, //上传并发数
    });

    // 当有文件被添加进队列的时候
    uploader.on('fileQueued', function (file) {
      $('#list').append('<blockquote class="layui-elem-quote layui-quote-nm" style="font-size:12px" id="' + file.id + '">'
        + file.name + '<span style="color:red;margin-left:10px">等待上传...</span>'+
      '</blockquote>');
    });

    //点击上传按钮
    $('#ctlBtn').on('click', function () {
      uploader.upload();
    });

    // 文件上传过程中创建进度条实时显示。
    uploader.on('uploadProgress', function (file, percentage) {
      var $li = $('#' + file.id),
        $percent = $li.find('.progress .progress-bar');
      // 避免重复创建
      if (!$percent.length) {
        $percent = $('<div class="progress progress-striped active">' +
          '<div class="progress-bar" role="progressbar" style="width: 0%">' +
          '</div>' +
          '</div>').appendTo($li).find('.progress-bar');
      }

      $li.find('span').text('上传中');

      $percent.css('width', percentage * 100 + '%');
    });

    uploader.on('uploadSuccess', function (file,response) {
      console.log(response)
      
      if( allfile == ''){
        allfile = response.getid;
      }else{
        allfile = allfile+','+response.getid;
      }
      document.getElementById("file_id").value = allfile;
      $('#' + file.id).find('span').text('已上传');
    });

    uploader.on('uploadError', function (file) {
      $('#' + file.id).find('span').text('上传出错');
    });

    uploader.on('uploadComplete', function (file) {
      $('#' + file.id).find('.progress').fadeOut();
    });

  });
</script>
```

```js
<!--图片上传 -->
<!-- <script type="text/javascript" src="__common__/js/jquery.min.js"></script> -->
<script type="text/javascript" src="__common__/js/webuploader.min.js"></script>
<script type="text/javascript" src="__common__/js/webuploads.js"></script> 
<!--自己配置 -->
<script type="text/javascript" src="__common__/js/upload.js"></script>
```

相关js文件放在 public/static/common/js下



可放在Main.php
```php
/**
 * @msg: ///图片上传
 * @param {type}
 * @return:
 */
  public function uplodas()
  {
      header("Expires: Mon, 26 Jul 1997 05:00:00 GMT");
      header("Last-Modified: " . gmdate("D, d M Y H:i:s") . " GMT");
      header("Cache-Control: no-store, no-cache, must-revalidate");
      header("Cache-Control: post-check=0, pre-check=0", false);
      header("Pragma: no-cache");
      
      if ($_SERVER['REQUEST_METHOD'] == 'OPTIONS') {
          exit; // finish preflight CORS requests here
      }
      if (!empty($_REQUEST[ 'debug' ])) {
          $random = rand(0, intval($_REQUEST[ 'debug' ]));
          if ($random === 0) {
              header("HTTP/1.0 500 Internal Server Error");
              exit;
          }
      }
      
      // header("HTTP/1.0 500 Internal Server Error");
      // exit;
      // 5 minutes execution time
      @set_time_limit(5 * 60);
      // Uncomment this one to fake upload time
      // usleep(5000);
      // Settings  //DIRECTORY_SEPARATOR /分隔符
      // $targetDir = ini_get("upload_tmp_dir") . DIRECTORY_SEPARATOR . "plupload";
      $targetDir = ROOT_PATH . 'public/uploads'.DIRECTORY_SEPARATOR;
      $uploadDir = ROOT_PATH . 'public/uploads'.DIRECTORY_SEPARATOR.date('Ymd');
      
      $cleanupTargetDir = true; // Remove old files
      $maxFileAge = 5 * 3600; // Temp file age in seconds
      // Create target dir
      if (!file_exists($targetDir)) {
          @mkdir($targetDir);
      }
      // Create target dir
      if (!file_exists($uploadDir)) {
          @mkdir($uploadDir);
      }
      // Get a file name
      if (isset($_REQUEST["name"])) {
          $fileName = $_REQUEST["name"];
      } elseif (!empty($_FILES)) {
          $fileName = $_FILES["file"]["name"];
      } else {
          $fileName = uniqid("file_");
      }

      
      $oldName = $fileName;

      $filePath = $targetDir . DIRECTORY_SEPARATOR . $fileName;
      
      // $uploadPath = $uploadDir . DIRECTORY_SEPARATOR . $fileName;
      // Chunking might be enabled
      $chunk = isset($_REQUEST["chunk"]) ? intval($_REQUEST["chunk"]) : 0;
      $chunks = isset($_REQUEST["chunks"]) ? intval($_REQUEST["chunks"]) : 1;
      // Remove old temp files
      if ($cleanupTargetDir) {
          if (!is_dir($targetDir) || !$dir = opendir($targetDir)) {
              die('{"jsonrpc" : "2.0", "error" : {"code": 100, "message": "Failed to open temp directory."}, "id" : "id"}');
          }
          while (($file = readdir($dir)) !== false) {
              $tmpfilePath = $targetDir . DIRECTORY_SEPARATOR . $file;
              // If temp file is current file proceed to the next
              if ($tmpfilePath == "{$filePath}_{$chunk}.part" || $tmpfilePath == "{$filePath}_{$chunk}.parttmp") {
                  continue;
              }
              // Remove temp file if it is older than the max age and is not the current file
              if (preg_match('/\.(part|parttmp)$/', $file) && (@filemtime($tmpfilePath) < time() - $maxFileAge)) {
                  @unlink($tmpfilePath);
              }
          }
          closedir($dir);
      }
      
      // Open temp file
      if (!$out = @fopen("{$filePath}_{$chunk}.parttmp", "wb")) {
          die('{"jsonrpc" : "2.0", "error" : {"code": 102, "message": "Failed to open output stream."}, "id" : "id"}');
      }
      

      if (!empty($_FILES)) {
          if ($_FILES["file"]["error"] || !is_uploaded_file($_FILES["file"]["tmp_name"])) {
              die('{"jsonrpc" : "2.0", "error" : {"code": 103, "message": "Failed to move uploaded file."}, "id" : "id"}');
          }
          // Read binary input stream and append it to temp file
          if (!$in = @fopen($_FILES["file"]["tmp_name"], "rb")) {
              die('{"jsonrpc" : "2.0", "error" : {"code": 101, "message": "Failed to open input stream."}, "id" : "id"}');
          }
      } else {
          if (!$in = @fopen("php://input", "rb")) {
              die('{"jsonrpc" : "2.0", "error" : {"code": 101, "message": "Failed to open input stream."}, "id" : "id"}');
          }
      }
      while ($buff = fread($in, 4096)) {
          fwrite($out, $buff);
      }
      @fclose($out);
      @fclose($in);
      rename("{$filePath}_{$chunk}.parttmp", "{$filePath}_{$chunk}.part");
      $index = 0;
      $done = true;
      for ($index = 0; $index < $chunks; $index++) {
          if (!file_exists("{$filePath}_{$index}.part")) {
              $done = false;
              break;
          }
      }

      
      if ($done) {
          $pathInfo = pathinfo($fileName);
          
          // $hashStr = substr(md5($pathInfo['basename']),8,16);
          //  $hashName = time() . $hashStr . '.' .$pathInfo['extension'];
          $rand = md5(time() . mt_rand(0, 1000));
          $hashName = $rand . '.' .$pathInfo['extension']; //$pathInfo['extension']图片格式

          //  $uploadPath = $uploadDir . DIRECTORY_SEPARATOR .$hashName; //原来随机路径
          $uploadPath = $uploadDir . DIRECTORY_SEPARATOR .$oldName;

          //插入数据库
          $data = array();
          $type = 'image/'.$pathInfo['extension'];
          $savepath = date("Ymd") .'/';
          $datas = ['name'=>$oldName,'img_type'=>$type,'savename'=>$hashName,'savepath'=>$savepath];
          $getid  = Db::name('file_list')->insertGetId($datas);
          
          //$sql = "INSERT INTO `add` (title) VALUES ('".$hashName."');";
          //$ok = $pdo->query($sql);   //第二种写法
                      
          // 打开文件写入
          if (!$out = @fopen($uploadPath, "wb")) {
              die('{"jsonrpc" : "2.0", "error" : {"code": 102, "message": "Failed to open output stream."}, "id" : "id"}');
          }

          if (flock($out, LOCK_EX)) {
              for ($index = 0; $index < $chunks; $index++) {
                  if (!$in = @fopen("{$filePath}_{$index}.part", "rb")) {
                      break;
                  }
                  while ($buff = fread($in, 4096)) {
                      fwrite($out, $buff);
                  }
                  @fclose($in);
                  @unlink("{$filePath}_{$index}.part");
              }
              flock($out, LOCK_UN);
          }
          @fclose($out);
          $response = [
          'success'=>true,
          'oldName'=>$oldName,
          'filePaht'=>$uploadPath,
          //'fileSize'=>$data['size'],
          'fileSuffixes'=>$pathInfo['extension'],
          //'file_id'=>$data['id'],
          'getid'=>$getid,
          ];
  
          die(json_encode($response));
      }
      
      // Return Success JSON-RPC response
      die('{"jsonrpc" : "2.0", "result" : null, "id" : "id"}');
  }
```

```php   
/**
   * @msg: 下载文件
  * @param {type} 
  * @return: 
  */
  public function download()
{
  $id = input('param.id');
  $file = Db::name('file_list')->where('id','eq',$id)->find();
  // $file_name = $file['savepath'].$file['savename'];
  $file_name = $file['savepath'].$file['name'];
  $file_dir =	ROOT_PATH.'public'.DS.'uploads'."/".$file_name;
  if(!file_exists($file_dir)){
    $this->error('文件找不到!');
  }else{
    //打开文件
    $file1 = fopen($file_dir,'r');
    //输入文件标签
    header("Content-type: application/octet-stream");
    header("Accept-Ranges: bytes");
    header("Accept-Length: ".filesize($file_dir));
    // header("Content-Disposition: attachment; filename=".$file['savename']);
    header("Content-Disposition: attachment; filename=".$file['name']);
    //输出文件内容
    echo fread($file1,filesize($file_dir));
    fclose($file1);
    exit();
  }

}
```
