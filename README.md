# UploadDownloadPDF
Many times, we need to work with the file and storing the physical files on the Server, which is very difficult because it will consume 
lots of physical hard disc space of the Server. Thus, in this article, we will learn, how to upload and download the files directly from the database in ASP.NET MVC. Thus, let's learn step by step so the beginners can also understand. 
Step 1 - Create MVC Application.   Now, let us start with a step by step approach from the creation of a simple MVC Application in the 
following- "Start", followed by "All Programs" and select "Microsoft Visual Studio 2015". 
Click "File", followed by "New" and click "Project". Select "ASP.NET Web Application Template", provide the Project a name as you 
wish and click OK. After clicking, the following Window will appear,   

Step 2 - Create Model Class Now, let us create the model 
class file, named EmpModel.cs, by right clicking on Models folder and define the following properties in EmpModel.cs class and 
FileDetailsModel as: The code snippet of EmpModel.cs and FileDetailsModel .cs will look like- 
public class EmpModel      
{          
[Required]          
[DataType(DataType.Upload)]          
[Display(Name ="Select File")]          
public HttpPostedFileBase files { get; set; }      
}         
public class FileDetailsModel      
{          
public int Id { get; set; }          
[Display(Name = "Uploaded File")]          
public String FileName { get; set; }         
public byte[] FileContent { get; set; }            }   
Step 3 - Create Table and Stored Procedure  Now, create the stored procedure and the table to store the uploaded files in binary 
format and display back to the user's view. Use the script, given below, to create the table named FileDetails as- 

CREATE TABLE [dbo].[FileDetails](       
[Id] [int] IDENTITY(1,1) NOT NULL,       
[FileName] [varchar](60) NULL,       
[FileContent] [varbinary](max) NULL   ) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY] 

As mentioned in the preceding table script, we have created three columns Id, which is to identify the files unique key. 
FileName to store uploaded file name and FileContent to store the uploaded file contents in the binary format.  
Create the stored procedure to insert the file details, using the script, given below- 

Create Procedure [dbo].[AddFileDetails]   
(   @FileName varchar(60),   @FileContent varBinary(Max)   )   
as   begin   Set NoCount on   Insert into FileDetails values(@FileName,@FileContent)      
End   

To get the uploaded file details, use the code, given below- 

CREATE Procedure [dbo].[GetFileDetails]   (   @Id int=null         )   
as   begin      select Id,FileName,FileContent from FileDetails   
where Id=isnull(@Id,Id)   End   We have created the tables and stored procedures. 

I hope, you have created the same. 

Step 4 - Add Controller Class Now, let us add ASP.NET MVC controller, as shown in the screenshot, given below-    
After clicking Add button, it will show in the Window. Specify the Controller name as Home with suffix Controller. 
Now, let's modify the default code of Home controller . After modifying the code of Homecontroller class, the code will look like- 

HomeController.cs     
using FileUploadDownLoadInMVC.Models;       
using System;       
using System.Collections.Generic;       
using System.IO;       
using System.Linq;       
using System.Web;       
using System.Web.Mvc;       
using Dapper;       
using System.Configuration;       
using System.Data.SqlClient;       
using System.Data;              
namespace FileUploadDownLoadInMVC.Controllers       
{           
public class HomeController : Controller           
{                             
#region Upload Download file               
public ActionResult FileUpload()               
{                   
return View();               
}                                    
[HttpPost]               
public ActionResult FileUpload(HttpPostedFileBase files)               
{                       
String FileExt=Path.GetExtension(files.FileName).ToUpper();                          
if (FileExt == ".PDF")                   {                       
Stream str = files.InputStream;                       
BinaryReader Br = new BinaryReader(str);                       
Byte[] FileDet = Br.ReadBytes((Int32)str.Length);                              
FileDetailsModel Fd = new Models.FileDetailsModel();                       
Fd.FileName = files.FileName;                       
Fd.FileContent = FileDet;                      
SaveFileDetails(Fd);                       
return RedirectToAction("FileUpload");                   
}                   
else                   
{                              
ViewBag.FileStatus = "Invalid file format.";                       
return View();                          }                                 
}                             
[HttpGet]               
public FileResult DownLoadFile(int id)               
{                                 
List&lt;FileDetailsModel> ObjFiles = GetFileList();                          
var FileById = (from FC in ObjFiles                                   
where FC.Id.Equals(id)                                   
select new { FC.FileName, FC.FileContent }).ToList().FirstOrDefault();                         
return File(FileById.FileContent, "application/pdf", FileById.FileName);                      }               
#endregion                     #region View Uploaded files               [HttpGet]               public PartialViewResult FileDetails()               {                   List&lt;FileDetailsModel> DetList = GetFileList();                          return PartialView("FileDetails", DetList);                             }               private List&lt;FileDetailsModel> GetFileList()               {                   List&lt;FileDetailsModel> DetList = new List&lt;FileDetailsModel>();                          DbConnection();                   con.Open();                   DetList = SqlMapper.Query&lt;FileDetailsModel>(con, "GetFileDetails", commandType: CommandType.StoredProcedure).ToList();                   con.Close();                   return DetList;               }                     #endregion                     #region Database related operations               private void SaveFileDetails(FileDetailsModel objDet)               {                          DynamicParameters Parm = new DynamicParameters();                   Parm.Add("@FileName", objDet.FileName);                   Parm.Add("@FileContent", objDet.FileContent);                   DbConnection();                   con.Open();                   con.Execute("AddFileDetails", Parm, commandType: System.Data.CommandType.StoredProcedure);                   con.Close();                             }               #endregion                     #region Database connection                      private SqlConnection con;               private string constr;               private void DbConnection()               {                    constr =ConfigurationManager.ConnectionStrings["dbcon"].ToString();                    con = new SqlConnection(constr);                      }               #endregion           }       }    The preceding code snippet explained everything to upload and download PDF files from the database. I hope, you have followed the same. Step 5 - Create strongly typed View Right click on View folder of the created Application and create two strongly typed views; one is to upload the files by choosing EmpModel.cs class  and Partial View by choosing FileDetailsModel class to display the uploaded files. The code snippet of the view looks like-  FileUpload.cshtml @model FileUploadDownLoadInMVC.Models.EmpModel      @{       ViewBag.Title = "www.compilemode.com";   }      @using (Html.BeginForm("FileUpload", "Home", FormMethod.Post, new { enctype = "multipart/form-data" }))   {          @Html.AntiForgeryToken()          &lt;div class="form-horizontal">           &lt;hr />           @Html.ValidationSummary(true, "", new { @class = "text-danger" })           &lt;div class="form-group">               @Html.LabelFor(model => model.files, htmlAttributes: new { @class = "control-label col-md-2" })               &lt;div class="col-md-10">                   @Html.TextBoxFor(model => model.files, "", new { @type = "file", @multiple = "multiple" })                   @Html.ValidationMessageFor(model => model.files, "", new { @class = "text-danger" })               &lt;/div>           &lt;/div>           &lt;div class="form-group">               &lt;div class="col-md-offset-2 col-md-10">                   &lt;input type="submit" value="Upload" class="btn btn-primary" />               &lt;/div>           &lt;/div>           &lt;div class="form-group">               &lt;div class="col-md-offset-2 col-md-10 text-success">                   @ViewBag.FileStatus               &lt;/div>           &lt;/div>              &lt;div class="form-group">               &lt;div class="col-md-8">                   @Html.Action("FileDetails", "Home")                  &lt;/div>           &lt;/div>       &lt;/div>   }      &lt;script src="~/Scripts/jquery-1.10.2.min.js">&lt;/script>   &lt;script src="~/Scripts/jquery.validate.min.js">&lt;/script>   &lt;script src="~/Scripts/jquery.validate.unobtrusive.min.js">&lt;/script>   FileDetails.cshtml @model IEnumerable&lt;FileUploadDownLoadInMVC.Models.FileDetailsModel>         &lt;table class="table table-bordered">       &lt;tr>           &lt;th class="col-md-4">               @Html.DisplayNameFor(model => model.FileName)           &lt;/th>                      &lt;th class="col-md-2">&lt;/th>       &lt;/tr>      @foreach (var item in Model) {       &lt;tr>           &lt;td>               @Html.DisplayFor(modelItem => item.FileName)           &lt;/td>                      &lt;td>               @Html.ActionLink("Downlaod", "DownLoadFile", new { id=item.Id })                          &lt;/td>       &lt;/tr>   }      &lt;/table>  Now, we have done all the coding.  Step 6 - Run the Application After running the Application, the UI of the Application will look like as follows-    Now select PDF file from your system and click Upload button. It will upload the file in the database and display back to the view is as follows-   Now, see the image, given below, of the table, which shows how the preceding uploaded file data is stored as-  From the preceding image, its clear that our uploaded file is stored into the database in the binary format. Now, click download button and it will show the following popup as-  Now, choose, whether to open the file or save the file according to your convenience. After opening the file, it will show the following contents, based on the uploaded file as-   I hope, from the preceding examples, you have learned, how to upload and download PDF files from the database In ASP.NET MVC, using FileResult.    Note  Its important to define enctype = "multipart/form-data" in form action, else the value will be null in the controller. Makes changes in web.config file connectionstring tag, based on your database location and configuration. Since this is a demo, it might not be using the proper standards. Thus, improve it, depending on your skills.
Many times, we need to work with the file and storing the physical files on the Server, which is very difficult because it will consume lots of physical hard disc space of the Server. Thus, in this article, we will learn, how to upload and download the files directly from the database in ASP.NET MVC. Thus, let's learn step by step so the beginners can also understand.
Step 1 - Create MVC Application.  
Now, let us start with a step by step approach from the creation of a simple MVC Application in the following-
"Start", followed by "All Programs" and select "Microsoft Visual Studio 2015".
Click "File", followed by "New" and click "Project". Select "ASP.NET Web Application Template", provide the Project a name as you wish 
and click OK. After clicking, the following Window will appear,


Step 2 - Create Model Class
Now, let us create the model class file, named EmpModel.cs, by right clicking on Models folder and define the following properties in EmpModel.cs class and FileDetailsModel as:
The code snippet of EmpModel.cs and FileDetailsModel .cs will look like-
public class EmpModel  
   {  
       [Required]  
       [DataType(DataType.Upload)]  
       [Display(Name ="Select File")]  
       public HttpPostedFileBase files { get; set; }  
   }  
  
   public class FileDetailsModel  
   {  
       public int Id { get; set; }  
       [Display(Name = "Uploaded File")]  
       public String FileName { get; set; }  
       public byte[] FileContent { get; set; }  
  
  
   }  
Step 3 - Create Table and Stored Procedure 
Now, create the stored procedure and the table to store the uploaded files in binary format and display back to the user's view. Use the script, given below, to create the table named FileDetails as-
CREATE TABLE [dbo].[FileDetails](  
    [Id] [int] IDENTITY(1,1) NOT NULL,  
    [FileName] [varchar](60) NULL,  
    [FileContent] [varbinary](max) NULL  
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
As mentioned in the preceding table script, we have created three columns Id, which is to identify the files unique key. FileName to store uploaded file name and FileContent to store the uploaded file contents in the binary format. 
Create the stored procedure to insert the file details, using the script, given below-
Create Procedure [dbo].[AddFileDetails]  
(  
@FileName varchar(60),  
@FileContent varBinary(Max)  
)  
as  
begin  
Set NoCount on  
Insert into FileDetails values(@FileName,@FileContent)  
  
End  
To get the uploaded file details, use the code, given below-
CREATE Procedure [dbo].[GetFileDetails]  
(  
@Id int=null  
  
  
)  
as  
begin  
  
select Id,FileName,FileContent from FileDetails  
where Id=isnull(@Id,Id)  
End  
We have created the tables and stored procedures. I hope, you have created the same.
Step 4 - Add Controller Class
Now, let us add ASP.NET MVC controller, as shown in the screenshot, given below-



After clicking Add button, it will show in the Window. Specify the Controller name as Home with suffix Controller. Now, let's modify the default code of Home controller . After modifying the code of Homecontroller class, the code will look like-
HomeController.cs
    using FileUploadDownLoadInMVC.Models;  
    using System;  
    using System.Collections.Generic;  
    using System.IO;  
    using System.Linq;  
    using System.Web;  
    using System.Web.Mvc;  
    using Dapper;  
    using System.Configuration;  
    using System.Data.SqlClient;  
    using System.Data;  
      
    namespace FileUploadDownLoadInMVC.Controllers  
    {  
        public class HomeController : Controller  
        {  
             
            #region Upload Download file  
            public ActionResult FileUpload()  
            {  
                return View();  
            }  
      
             
            [HttpPost]  
            public ActionResult FileUpload(HttpPostedFileBase files)  
            {  
      
             String FileExt=Path.GetExtension(files.FileName).ToUpper();  
      
                if (FileExt == ".PDF")  
                {  
                    Stream str = files.InputStream;  
                    BinaryReader Br = new BinaryReader(str);  
                    Byte[] FileDet = Br.ReadBytes((Int32)str.Length);  
      
                    FileDetailsModel Fd = new Models.FileDetailsModel();  
                    Fd.FileName = files.FileName;  
                    Fd.FileContent = FileDet;  
                    SaveFileDetails(Fd);  
                    return RedirectToAction("FileUpload");  
                }  
                else  
                {  
      
                    ViewBag.FileStatus = "Invalid file format.";  
                    return View();  
      
                }  
                 
            }  
             
            [HttpGet]  
            public FileResult DownLoadFile(int id)  
            {  
      
      
                List<FileDetailsModel> ObjFiles = GetFileList();  
      
                var FileById = (from FC in ObjFiles  
                                where FC.Id.Equals(id)  
                                select new { FC.FileName, FC.FileContent }).ToList().FirstOrDefault();  
      
                return File(FileById.FileContent, "application/pdf", FileById.FileName);  
      
            }  
            #endregion  
     
            #region View Uploaded files  
            [HttpGet]  
            public PartialViewResult FileDetails()  
            {  
                List<FileDetailsModel> DetList = GetFileList();  
      
                return PartialView("FileDetails", DetList);  
      
      
            }  
            private List<FileDetailsModel> GetFileList()  
            {  
                List<FileDetailsModel> DetList = new List<FileDetailsModel>();  
      
                DbConnection();  
                con.Open();  
                DetList = SqlMapper.Query<FileDetailsModel>(con, "GetFileDetails", commandType: CommandType.StoredProcedure).ToList();  
                con.Close();  
                return DetList;  
            }  
     
            #endregion  
     
            #region Database related operations  
            private void SaveFileDetails(FileDetailsModel objDet)  
            {  
      
                DynamicParameters Parm = new DynamicParameters();  
                Parm.Add("@FileName", objDet.FileName);  
                Parm.Add("@FileContent", objDet.FileContent);  
                DbConnection();  
                con.Open();  
                con.Execute("AddFileDetails", Parm, commandType: System.Data.CommandType.StoredProcedure);  
                con.Close();  
      
      
            }  
            #endregion  
     
            #region Database connection  
      
            private SqlConnection con;  
            private string constr;  
            private void DbConnection()  
            {  
                 constr =ConfigurationManager.ConnectionStrings["dbcon"].ToString();  
                 con = new SqlConnection(constr);  
      
            }  
            #endregion  
        }  
    }   
The preceding code snippet explained everything to upload and download PDF files from the database. I hope, you have followed the same.
Step 5 - Create strongly typed View
Right click on View folder of the created Application and create two strongly typed views; one is to upload the files by choosing EmpModel.cs class  and Partial View by choosing FileDetailsModel class to display the uploaded files. The code snippet of the view looks like-

FileUpload.cshtml
@model FileUploadDownLoadInMVC.Models.EmpModel  
  
@{  
    ViewBag.Title = "www.compilemode.com";  
}  
  
@using (Html.BeginForm("FileUpload", "Home", FormMethod.Post, new { enctype = "multipart/form-data" }))  
{  
  
    @Html.AntiForgeryToken()  
  
    <div class="form-horizontal">  
        <hr />  
        @Html.ValidationSummary(true, "", new { @class = "text-danger" })  
        <div class="form-group">  
            @Html.LabelFor(model => model.files, htmlAttributes: new { @class = "control-label col-md-2" })  
            <div class="col-md-10">  
                @Html.TextBoxFor(model => model.files, "", new { @type = "file", @multiple = "multiple" })  
                @Html.ValidationMessageFor(model => model.files, "", new { @class = "text-danger" })  
            </div>  
        </div>  
        <div class="form-group">  
            <div class="col-md-offset-2 col-md-10">  
                <input type="submit" value="Upload" class="btn btn-primary" />  
            </div>  
        </div>  
        <div class="form-group">  
            <div class="col-md-offset-2 col-md-10 text-success">  
                @ViewBag.FileStatus  
            </div>  
        </div>  
  
        <div class="form-group">  
            <div class="col-md-8">  
                @Html.Action("FileDetails", "Home")  
  
            </div>  
        </div>  
    </div>  
}  
  
<script src="~/Scripts/jquery-1.10.2.min.js"></script>  
<script src="~/Scripts/jquery.validate.min.js"></script>  
<script src="~/Scripts/jquery.validate.unobtrusive.min.js"></script>  
FileDetails.cshtml
@model IEnumerable<FileUploadDownLoadInMVC.Models.FileDetailsModel>  
  
  
<table class="table table-bordered">  
    <tr>  
        <th class="col-md-4">  
            @Html.DisplayNameFor(model => model.FileName)  
        </th>  
          
        <th class="col-md-2"></th>  
    </tr>  
  
@foreach (var item in Model) {  
    <tr>  
        <td>  
            @Html.DisplayFor(modelItem => item.FileName)  
        </td>  
          
        <td>  
            @Html.ActionLink("Downlaod", "DownLoadFile", new { id=item.Id })   
             
        </td>  
    </tr>  
}  
  
</table> 
Now, we have done all the coding.

Step 6 - Run the Application
After running the Application, the UI of the Application will look like as follows-



Now select PDF file from your system and click Upload button. It will upload the file in the database and display back to the view is as follows-


Now, see the image, given below, of the table, which shows how the preceding uploaded file data is stored as-

From the preceding image, its clear that our uploaded file is stored into the database in the binary format. Now, click download button and it will show the following popup as-

Now, choose, whether to open the file or save the file according to your convenience. After opening the file, it will show the following contents, based on the uploaded file as-


I hope, from the preceding examples, you have learned, how to upload and download PDF files from the database In ASP.NET MVC, using FileResult.   
Note 
Its important to define enctype = "multipart/form-data" in form action, else the value will be null in the controller.
Makes changes in web.config file connectionstring tag, based on your database location and configuration.
Since this is a demo, it might not be using the proper standards. Thus, improve it, depending on your skills.
