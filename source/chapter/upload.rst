第十四章 文件上传
===============

使用表单上传文件
---------------

标准的文件上传方式是通过表单形式来上传，HTML表单如下：

.. code-block:: scala
  @form(action = routes.Application.upload, 'enctype -> "multipart/form-data") {

      <input type="file" name="picture">

      <p>
          <input type="submit">
      </p>

  }

后台上传代码如下：

.. code-block:: scala
  
  public Result upload() {
      MultipartFormData<File> body = request().body().asMultipartFormData();
      FilePart<File> picture = body.getFile("picture");
      if (picture != null) {
          String fileName = picture.getFilename();
          String contentType = picture.getContentType();
          File file = picture.getFile();
          return ok("File uploaded");
      } else {
          flash("error", "Missing file");
          return badRequest();
      }
  }


直接文件上传
-----------

另一种方式是通过JSON方式上传：

.. code-block:: scala
  
  public Result upload() {
      File file = request().body().asRaw().asFile();
      return ok("File uploaded");
  }
