The Front End project was the third project during my time at the Tech Academy.

There are total 4 Live Projects offered by the Academy to its students: Python Project, C# Project, Front End and Back End Projects.Ussually Front End and Back End Projects  are done after Python and C# Sharp Projects. Students can select either Python or C# for their front-end and back-end projects.I selected C# (please also see C# Live Project) for the front-end.

About the project application:

The application was for a real client of The Tech Academy.
The users of the application were divided into 2 groups: Admins and other Users. Admins were able to create and 
distribute a weekly schedule assigning users to certain jobs. Other Users were able to keep track of which job they are 
assigned to for the week. The application also had the other features such as:  dashboard with company news and 
announcements, and chat functionality for the users to communicate. 
The project was done in ASP.NET MVC 5.

This time I was working mostly on front-end user stories:

#### 1.Confirmation messages
The application had a several confirmation messages that poped-up through out the program saying somthing like "LocalHostxxxx Says:...".
I needed to create a universal way of packaging these to be more UI friendly.  I used Bootstrap modals and JQuery to create the message pop-up.

Script for Confirmation Modal - how it works:

The script works for elements of a form. If an element is not a part of a form the element should be wrap in the form.
The modal should have a unique Id.
The content of the modal without message in the modal-body comes from ConfirmMessageModalPartial view.
Variable modConfirm declared in the script holds the function with 2 parameters: 
msg – message for the modal - body and formId - the Id of the form that should be submitted.
The Id of the form should be unique too therefore Guid.NewGuid method used to generate that Id outside of the form.
Then the Id is passed into the form through a string variable.

~~~
var modConfirm = function (msg, formId) { 
    event.preventDefault();
    $("#btnOK").unbind('click');
    $("#btnCancel").unbind('click');
    $("#btnOK").on("click", function () {
        $("#confirmModal").modal('hide');
        $("#" + formId).trigger('submit');//
    });
    $("#btnCancel").on("click", function () {
        $("#confirmModal").modal('hide');
    });
    $("#confirmModal .modal-body").text(msg);
    $("#confirmModal").modal('show');
    return false;
}
~~~
View (any view where the Confirmation Modal required):
~~~
<!---Confirmation Modal-->
    <div class="modal fade" id="confirmModal" tabindex="-1" role="dialog" aria-hidden="true">
        @Html.Partial("~/Views/Shared/ConfirmMessageModalPartial.cshtml")
    </div>
<!--Confirmation Modal end-->
~~~
ConfirmMessageModalPartial view:
~~~
<!--The content of the ConfirmModal-->
<div class="modal-dialog" style="width:500px">
    <div class="modal-content" id="modalContent">
        <div class="modal-header text-center">
            <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                <span aria-hidden="true">&times;</span>
            </button>
            <h3 class="modal-title" id="exampleModalLabel" style="color:black">Confirmation</h3>
        </div>
        <div class="modal-body text-center" style="font-size:16px">

        </div>
        <div class="modal-footer">
            <button type="button" class="btn btn-primary" id="btnOK" style="width:75px">OK</button>
            <button type="button" class="btn btn-default" data-dismiss="modal" id="btnCancel" style="width:75px">Cancel</button>
        </div>
    </div>
</div>
~~~
After creating the modal I needed to implement it through the application without removing any functionality of the project.
The sample code is below:
~~~
 <td>
                @{ string editworkFormId = Guid.NewGuid().ToString();}
                @using (Html.BeginForm("EditWork", "Account", FormMethod.Post, new { enctype = "multipart/form-data", id = editworkFormId }))
                {
                    @Html.Hidden("Id", item.Id)
                    @Html.DropDownList("workType", new SelectList(ConstructionNew.Extensions.WorkTypeMethods.GetWorkTypeDescription()), ConstructionNew.Extensions.EnumExtensions.GetDescription(item.WorkType))
                    <input type="submit" value="Submit" onclick="modConfirm('Click ok to change category','@(editworkFormId)')" />
                }
            </td>
            <td>
                @{ string editroleFormId = Guid.NewGuid().ToString();}
                @using (Html.BeginForm("EditRole", "Account", FormMethod.Post, new { enctype = "multipart/form-data", id = editroleFormId }))
                {
                    @Html.Hidden("Id", item.Id)
                    @Html.DropDownList("UserRole", new List<SelectListItem>
                    {
                        new SelectListItem { Text = "Admin", Value = "Admin"},
                        new SelectListItem { Text = "Manager", Value = "Manager"},
                        new SelectListItem { Text = "Employee", Value = "Employee"}
                    }, item.UserRole)
                    <input type="submit" value="Submit" onclick="modConfirm('Click Ok to change role','@(editroleFormId)')" />
                }
            </td>
            <td>
                <!--Creates form for Suspend user and ties it to the "Suspend" method in the "AccountController" -->
                @{ string suspendFormId = Guid.NewGuid().ToString();}
                @using (Html.BeginForm("Suspend", "Account", FormMethod.Post, new { enctype = "multipart/form-data", id = suspendFormId }))
                {
                    <!--Creates checkbox for Suspend and binds it to the model attribute "Suspended"-->
                    @Html.Hidden("Id", item.Id)
                    @Html.CheckBoxFor(model => item.Suspended, new { Name = "suspended" })

                    // submit button for users with current and future schedules
                    if (ViewBag.currentFutureUsers.Contains(item))
                    {
                        <input type="submit" value="Submit" onclick="modConfirm('Suspend user and remove all assigned schedules?','@(suspendFormId)')" />
                    }
                    //submit button for users that are currently suspended
                    else if (item.Suspended)
                    {
                        <input type="submit" value="Submit" onclick="modConfirm('Are you sure you want to unsuspend this user?','@(suspendFormId)')" />
                    }
                    //submit button for unsuspended users without current or future schedules
                    else
                    {
                        <input type="submit" value="Submit" onclick="modConfirm('Are you sure you want to suspend this user?','@(suspendFormId)')" />
                    }
                }
            </td>
            <td>
                @{ string deleteFormId = Guid.NewGuid().ToString();}
                @using (Html.BeginForm("Delete", "Account", FormMethod.Post, new { enctype = "multipart/form-data", id = deleteFormId }))
                {
                    @Html.Hidden("Id", item.Id)
                    <a href="" onclick="modConfirm('Press Ok to delete','@(deleteFormId)')">Delete User</a>
                }
            </td>
~~~
#### 2.Edit and Delete Company News
Admin could create Company News from the dashboard. If Admin wanted to edit or delete any news they had to click edit or delete buttons and they would be redirected to the different Edit or Delete pages of the application. 
Using the Bootstrap modal and Ajax I implemented a pop up modal for editing Company News. And used the confirmation modal above to confirm the news deletion.The redirecting pages were no longer required.

Dashboard View (where the Edit News Modal should pop-up):
~~~
 <!--Edit Company News Modal-->
    <div class="modal fade" id="editNewsModal" tabindex="-1" role="dialog" aria-hidden="true">
        <div class="modal-dialog" style="width:630px">
            <div class="modal-content" id="editNewsContainer">

            </div>
        </div>
    </div>
 <!--Edit Company News Modal end-->
~~~
EditCompanyNewsPartialView ( the content of the modal):
~~~
@model ConstructionNew.Models.CompanyNews

<div class="modal-header text-center">
    <button type="button" class="close" data-dismiss="modal" aria-label="Close">
        <span aria-hidden="true">&times;</span>
    </button>
    <h3 class="modal-title" id="exampleModalLabel" style="color:black">Edit Company News</h3>
</div>

    @using (Ajax.BeginForm("Edit", "CompanyNews", FormMethod.Post,
                        new AjaxOptions
                        {
                            InsertionMode = InsertionMode.Replace,
                            HttpMethod = "POST",
                            UpdateTargetId = "collapseCompanyNews"
                        }))
    {

        @Html.AntiForgeryToken()

        @Html.ValidationSummary(true, "", new { @class = "text-danger" })
        @Html.HiddenFor(model => model.DateStamp)
        <div class="modal-body">

            <div class="form-group  modal-news">
                @Html.LabelFor(model => model.Title, htmlAttributes: new { @class = "control-label-lg" })
                <br />
                @Html.EditorFor(model => model.Title, new { htmlAttributes = new { @class = "form-control" } })
                @Html.ValidationMessageFor(model => model.NewsItem, "", new { @class = "text-danger" })
            </div>
            <div class="form-group  modal-news">
                @Html.LabelFor(model => model.NewsItem, htmlAttributes: new { @class = "control-label" })
                <br />
                @Html.EditorFor(model => model.NewsItem, new { htmlAttributes = new { @class = "form-control" } })
                @Html.ValidationMessageFor(model => model.NewsItem, "", new { @class = "text-danger" })
            </div>
            <div class="form-group modal-news">
                @Html.LabelFor(model => model.ExpirationDate, htmlAttributes: new { @class = "date" })
                <br />
                @Html.EditorFor(model => model.ExpirationDate, new { htmlAttributes = new { @class = "datepicker", type = "date" } })
                @Html.ValidationMessageFor(model => model.ExpirationDate, "", new { @class = "text-danger" })
            </div>
        </div>
        <div class="modal-footer companyNewsFooter">
            <button type="button" class="btn btn-default" data-dismiss="modal">Close</button>
            <button type="submit" class="btn btn-primary">Save changes</button>
        </div>
}
~~~
Script  for Edit Company News:
~~~

       var editNews = function (newsID) {
            var url = "/CompanyNews/Edit"; // the url to the controller
            $.get(url + '/' + newsID, function (data) {
                $('#editNewsContainer').html(data);
                $('#editNewsModal').modal('show');
            });
        }

~~~
This is how the Delete Company News Modal called:
~~~

@{ string deleteFormId = Guid.NewGuid().ToString();}
@using (Html.BeginForm("Delete", "CompanyNews", FormMethod.Post, new { enctype = "", id = deleteFormId }))
          {
                @Html.AntiForgeryToken()
                @Html.Hidden("Id", item.DateStamp)
                <a class="icon-spacing" href="" onclick="modConfirm('Press Ok to delete','@(deleteFormId)')">
                   <span class="glyphicon glyphicon-trash" aria-hidden="true"></span>
                </a>
           }                                

~~~

#### 3.Dashboard- Users section look
Added a title for the different sections of users (Active and Suspended) in the dashboard and created a zebra striping look the users table-lists. 

Dashboard View:
~~~
<div id="collapseUserList" class="panel-collapse collapse show" role="tabpanel" aria-labelledby="headingTwo">
               <h4 class="panel-heading">
                   Active Users
               </h4>
               <div class="panel-body table-style">
                  @Html.Action("_UserListPartial", "Account")
               </div>
</div>
<div id="collapseUserList" class="panel-collapse collapse show" role="tabpanel" aria-labelledby="headingTwo">          
               <h4 class="panel-heading">
                   Suspended Users
                    </h4>
               <div class="panel-body table-style suspended-style">                      
                    @Html.Action("_SuspendedUserPartial", "Account")
	       </div>   
</div>
~~~
CSS:
~~~
div.table-style tr:nth-child(even) {
    background-color: lightgrey;
}

div.table-style tr {
    border: hidden;
}

h4.panel-heading {
    font-weight: bold;
}

div.suspended-style {
    border-top: 1px solid #ddd;
}
~~~
#### 4.Message for the users with no schedules
Created a partial view that let the user know they were not on the schedule. Before the user could see the blank page only.

NullSchedulePartial view:
~~~
<div class="alert alert-light" role="alert" style="padding-left:unset">
    <h4>Currently you are not on the schedule. If you think this is an error please contact your supervisor.</h4>
</div>
~~~
MySchedulePartial view ( extract):
~~~
@if ((countUserSchedules == 0) && (User.IsInRole("Employee")))
    {
        @Html.Partial("~/Views/Schedules/NullSchedulePartial.cshtml")
    }
~~~
