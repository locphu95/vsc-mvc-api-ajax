# vsc-mvc-api-ajax
$(function () {
    init();
})

function init() {
    var userObjectId = getThisCookie(userObjectIdCookie);
    var roleArr = [null, 'Admin', 'Editor'];
    var productListVM = {
        dt: null,

        init: function () {
            dt = $('#userTable').DataTable({
                "serverSide": true,
                //"processing": true,
                "ajax": {
                    "url": "/User/DataTableGet",
                    "type": "GET",
                    "data": { userObjectId: userObjectId }
                },
                'aaSorting': [[1, 'asc']], // start to sort data in second column
                "columns": [
                    //{
                    //    "title": "No", "data": null, "searchable": false, "sortable": false, "bSortable": false, "bSort": false
                    //},
                    { "title": "Display Name", "data": "DisplayName" },
                    { "title": "Email", "data": "AuthenticationEmail" },
                    //Tham-NTK del backlog 1898 start
                    //{ "title": "Job Title", "data": "JobTitle" },
                    //Tham-NTK del backlog 1898 end
                    { "title": "Department", "data": 'Department' },
                    {
                        "title": "Permission", "data": null,
                        mRender: function (data) {
                            var opt = '';
                            var activeOpt = '';
                            for (var i = 0; i < roleArr.length; i++) {
                                if (data.Role === roleArr[i]) {
                                    data.Role = roleArr[i];
                                    activeOpt = 'selected="selected"';
                                }
                                else {
                                    if (data.Role && roleArr[i] && data.Role.toLowerCase() == roleArr[i].toLowerCase()) {
                                        data.Role = roleArr[i];
                                        activeOpt = 'selected="selected"';
                                    }
                                    else
                                        activeOpt = '';
                                }
                                opt += '<option value="' + roleArr[i] + '" ' + activeOpt + '>' + (roleArr[i] == null ? 'Viewer' : roleArr[i]) + '</option>';
                            }
                            return '<select class="roles" oid="' + data.ObjectId + '" uname="' + data.UserName + '" uid="' + data.Id + '" email="' + data.AuthenticationEmail + '" role="' + data.Role + '">' + opt + '</select>';
                        }
                    },
                    // ADD Duc-BTT 20180307 Event作成権限付与画面 START
                    {
                        "title": "Event Owner", "data": null
                        , mRender: function (data) {
                            var isChecked = '';
                            if (data.IsEventOrganizer == 1)
                                isChecked = 'checked';
                            return '<input type="checkbox"' + isChecked + ' oid="' + data.ObjectId + '" uname="' + data.UserName + '" uid="' + data.Id + '" email="' + data.AuthenticationEmail + '" role="' + data.Role + '" class="event-owner" uowner="' + isChecked + '"/>';
                        }
                    },
                    // ADD Duc-BTT 20180307 Event作成権限付与画面 END
                    {
                        "title": "Status", "data": "Status", "searchable": false, "bSort": false, "sortable": false, "bSortable": false
                    }
                ],
                "lengthMenu": [[5, 10, 15], [5, 10, 15]],
                "fnCreatedRow": function (row, data, index) {
                    var renderStatus = data.Status == 1 ? '<input type="button" name="" class="btnActive btn btn-primary btn-sm" value="Active" />' : '<input type="button" name="" class="btnActive btn btn-info btn-sm" value="Inactive" />';
                    //Tham-NTK mod backlog 1898 start
                    // MOD Duc-BTT 20180307 Event作成権限付与画面 START
                    // $('td', row).eq(4).html(renderStatus);
                    $('td', row).eq(5).html(renderStatus);
                    // MOD Duc-BTT 20180307 Event作成権限付与画面 END
                    //Tham-NTK mod backlog 1898 end
                },
                "drawCallback": function (settings) {
                    $('input.btnActive').click(function () {
                        if (confirm(mimasResource.getResourceByKey('Confirm change status'))) {
                            $(this).prop('disabled', true);
                            var data = dt.row($(this).closest('tr')).data();
                            updateStatus(data, this);
                        }
                        else {
                            $(this).prop('disabled', false);
                        }
                    });
                }
            });
        },

        refresh: function () {
            dt.ajax.reload();
        }
    }

    $('#refresh-button').on("click", productListVM.refresh);

    if (!flag) {
        productListVM.init();
        flag = true;
    }
    else {
        productListVM.refresh();
    }
    // ADD Duc-BTT 20180307 Event作成権限付与画面 START
    var isAdminTenant = getThisCookie('key_owner');
    if (isAdminTenant != 3)
        // Get the column API object
        dt.column(4).visible(false)
    // ADD Duc-BTT 20180307 Event作成権限付与画面 END
}
var flag = false;
// ADD Duc-BTT 20180307 Event作成権限付与画面 START
function updateIsEventOrganizer(checkbox) {
    var userObjectId = getThisCookie(userObjectIdCookie);
    $.post('/User/UpdateIsEventOrganizer', { id: $(checkbox).attr("uid"), userObjectId: userObjectId, isEventOrganizer: $(checkbox).is(":checked") }).then(function (response) {
        if (response.Success) {
            MessageShowHide('alert-success', mimasResource.getResourceByKey('updateSuccessfull'), 5000);
        } else {
            MessageShowHide('alert-warning', mimasResource.getResourceByKey('updateFail'), 5000);
        }
    }, function (xhr, status, error) {
        MessageShowHide('alert-warning', mimasResource.getResourceByKey('updateFail'), 5000);
    });
}
// ADD Duc-BTT 20180307 Event作成権限付与画面 END
function updateStatus(data, btnActive) {
    var userObjectId = getThisCookie(userObjectIdCookie);
    $.post('/User/UpdateStatus', { id: data.Id, userObjectId: userObjectId }).then(function (response) {
        if (response.Success) {
            MessageShowHide('alert-success', mimasResource.getResourceByKey('updateSuccessfull'), 5000);
            var renderStatus = response.Result.Status == 1 ? '<input type="button" name="" class="btnActive btn btn-primary btn-sm" value="Active" />' : '<input type="button" name="" class="btnActive btn btn-info btn-sm" value="Inactive" />';
            var td = $(btnActive).parent();
            td.html(renderStatus);
            $(td.children()[0]).click(function () {
                if (confirm(mimasResource.getResourceByKey('Confirm change status'))) {
                    $(this).prop('disabled', true);
                    var data = dt.row($(this).closest('tr')).data();
                    updateStatus(data, this);
                }
                else {
                    $(this).prop('disabled', false);
                }
            });
            //init();
        }
        else {
            MessageShowHide('alert-warning', mimasResource.getResourceByKey('updateFail'), 5000);
            $(btnActive).prop('disabled', false);
        }
    }, function (xhr, status, error) {
        alert('There are error occur, please contact administrator.');
    });
}

function confirmPermission() {
    var tr = '';
    // MOD Duc-BTT 20180307 Event作成権限付与画面 START
    //$('.roles').each(function () {
    //    var newRole = $(this).val();
    //    var originRole = $(this).attr('role');
    //    if (originRole != newRole || (originRole && newRole && originRole.toLowerCase() != newRole.toLowerCase())) {
    //        var x = this.selectedIndex;
    //        var y = this.options;
    //        var displayRole = y[x].text;
    //        var email = $(this).attr('email');
    //        var uid = $(this).attr('uid');
    //        var uname = $(this).attr('uname');
    //        var oid = $(this).attr('oid');
    //        tr += '<tr class="newRoleUsers" oid="' + oid + '" email="' + email + '" role="' + newRole + '" uname="' + uname + '" uid="' + uid + '"><td>' + email + '</td><td>' + displayRole + '</td></tr>';
    //    }
    //});
    $('#userTable tbody tr').each(function (val, idx) {
        var roleElement = $(this).find(".roles");
        var originRole = $(roleElement).attr("role");
        var newRole = $(roleElement).val();

        var eventOwnerElement = $(this).find(".event-owner");
        var originEventOwner = $(eventOwnerElement).attr("uowner") == "checked";
        var newEventOwner = $(eventOwnerElement).is(":checked");

        var isChecked = '';
        if (newEventOwner)
            isChecked = 'checked';

        if ((originRole != newRole || (originRole && newRole && originRole.toLowerCase() != newRole.toLowerCase()))
            || originEventOwner != newEventOwner) {
            var displayRole = $(roleElement).find(":selected").html();
            var email = $(roleElement).attr('email');
            var uid = $(roleElement).attr('uid');
            var uname = $(roleElement).attr('uname');
            var oid = $(roleElement).attr('oid');
            tr += '<tr class="newRoleUsers" oid="' + oid + '" email="' + email + '" role="' + newRole + '" uname="' + uname + '" uid="' + uid + '">';
            tr += '<td>' + email + '</td>';
            tr += '<td>' + displayRole + '</td>';
            tr += '<td><input type="checkbox" ' + isChecked + ' disabled/></td>';
            tr += '</tr>';
        }
    });
    // MOD Duc-BTT 20180307 Event作成権限付与画面 END
    $('#modalConfirm tbody').html(tr);
}

function loadProfile() {
    var userObjectId = getThisCookie(userObjectIdCookie);
    $.ajax({
        method: 'GET',
        url: '/User/GetProfile',
        data: { userObjectId: userObjectId }
    }).then(function (response) {
        if (response.Success) {
            var result = response.Result;
            if (result.DisplayName) {
                $('#displayname').val(result.DisplayName);
            }
            if (result.MobilePhone) {
                $('#telephone').val(result.MobilePhone);
            }
            if (result.Department) {
                $('#department').val(result.Department);
            }
            if (result.JobTitle) {
                $('#jobtitle').val(result.JobTitle);
            }
            if (result.Avatar) {
                $('.avatarSignup').prop('src', result.Avatar);
                imageOrigin = result.Avatar;
            }
            else {
                $(".avatarSignup").prop('src', '/App/img/uploadprofile.png');
            }
        }
        else {
            alert('There are error occur, please contact administrator.');
        }
    }, function (response) {
        alert('There are error occur, please contact administrator.');
    });
}
var imageOrigin = '';
function loadUserProfile() {
    var userObjectId = getThisCookie(userObjectIdCookie);
    $.ajax({
        method: 'GET',
        url: '/User/ViewUserProfile',
        data: { userObjectId: userObjectId }
    }).then(function (response) {
        if (response.Success) {
            var result = response.Result;

            if (result.DisplayName) {
                $('.profile-displayName').text(result.DisplayName);
            }
            if (result.MobilePhone) {
                $('.profile-mobile').text(result.MobilePhone);
            }
            if (result.AuthenticationEmail) {
                $('.profile-email').text(result.AuthenticationEmail);
            }
            if (result.Avatar) {
                $('.avatarSignup').prop('src', result.Avatar);
                imageOrigin = result.Avatar;
                //$("#filebrowse").prop("value", result.Avatar);
            }
            else {
                $(".avatarSignup").prop('src', '/App/img/uploadprofile.png');
            }
            if (result.Department) {
                $('.profile-department').text(result.Department);
            }
            if (result.JobTitle) {
                $('.profile-jobtitle').text(result.JobTitle);
            }
        }
        else {
            alert('There are error occur, please contact administrator.');
        }
    }, function (response) {
        alert('There are error occur, please contact administrator.');
    });
}

function showUpload() {
    $("#filebrowse").click();
}
function handleFileSelect(evt) {
    var files = evt.target.files; // FileList object

    if (files.length == 0) {
        $(".avatarSignup").attr('src', '/App/img/uploadprofile.png');
        return;
    }

    // Loop through the FileList and render image files as thumbnails.
    for (var i = 0, f; f = files[i]; i++) {
        // Only process image files.
        if (!f.type.match('image.*')) {
            continue;
        }

        var reader = new FileReader();

        // Closure to capture the file information.
        reader.onload = (function (theFile) {
            return function (e) {
                // Render thumbnail.
                $(".avatarSignup").attr('src', e.target.result);
            };
        })(f);

        // Read in the image file as a data URL.
        reader.readAsDataURL(f);
    }
}
function isValidProfile(user) {
    var arrayErrors = [];
    if (user.MobilePhone == '') {
        MessageShowHide('alert-warning-telephone', 'Mobile must not be blank', 7000);
        arrayErrors.push(1);
    }
    if (user.DisplayName == '') {
        MessageShowHide('alert-warning-displayname', 'Display Name must not be blank', 7000);
        arrayErrors.push(1);
    }
    if (user.MobilePhone != '') {
        var phonenoSign = /^\+?([0-9]{2})\)?[-. ]?([0-9]{2})\)?[-. ]?([0-9]{4})[-. ]?([0-9]{4})$/;
        if (document.formUpdateProfile.telephone.value.match(phonenoSign)) {
        }
        else {
            MessageShowHide('alert-warning-telephone', 'Not a valid Phone Number', 7000);
            arrayErrors.push(1);
        }
    }
    if (arrayErrors.length > 0) {
        return false;
    }
    return true;
}
function updateProfile() {
    $('#btnUpdateProfile').attr('disabled', 'disabled');
    var user = {
        MobilePhone: $("#telephone").val(),
        DisplayName: $("#displayname").val(),
        Department: $("#department").val(),
        JobTitle: $("#jobtitle").val()
    };
    if (!isValidProfile(user)) {
        $('#btnUpdateProfile').removeAttr('disabled');
        return;
    }
    if (window.FormData === undefined) {
        return;
    }
    var fileUpload = $("#filebrowse").get(0);
    var files = fileUpload.files;
    if (files.length > 0) {
        var fileData = new FormData();
        for (var i = 0; i < files.length; i++) {
            fileData.append(files[i].name, files[i]);
        }
        var url = '/api/Users/PostFile';
        $.ajax({
            url: url,
            type: 'POST',
            enctype: 'multipart/form-data',
            contentType: false,
            processData: false,
            data: fileData,
        }).then(PostFileSuccess, PostFileError);
    }
    else {
        var currentImage = $('.avatarSignup').attr('src');
        if (currentImage != imageOrigin) {
            PostFileSuccess({ data: { Image: '' } });
        }
        else {
            PostFileSuccess({ Image: imageOrigin });
        }
    }
    function PostFileSuccess(image) {
        user.Avatar = image.Image;
        user.ObjectId = getThisCookie(userObjectIdCookie);
        url = '/api/Users/UpdateProfile';
        $.post(url, user).then(UpdateProfileSuccess, UpdateProfileError);

        function UpdateProfileSuccess(response) {
            if (response.Success) {
                $('.user-profile img').prop('src', user.Avatar);
                $('.alert-success').text('You are updated successfully');
                $('.alert-success').show();
                setTimeout(function () {
                    window.location.href = '/user/userprofile';
                }, 5000);
            }
            else {
                alert('There are error occur, please contact administrator.');
                $('#btnUpdateProfile').removeAttr('disabled');
            }
        };
        function UpdateProfileError(e) {
            $('#btnUpdateProfile').removeAttr('disabled');
        }
    };
    function PostFileError(e) {
        $('#btnUpdateProfile').removeAttr('disabled');
    }
}
function isValidUpdatePwd(viewModel) {
    var arrayErrors = [];
    if (!viewModel.OldPwd) {
        MessageShowHide('alert-warning-oldPwd', 'You can’t leave this blank.', 5000);
        arrayErrors.push(1);
    }
    if (!viewModel.NewPwd) {
        MessageShowHide('alert-warning-newPwd', 'You can’t leave this blank.', 5000);
        arrayErrors.push(1);
    }
    if (!viewModel.ConfirmPwd) {
        MessageShowHide('alert-warning-confirmPwd', 'You can’t leave this blank.', 5000);
        arrayErrors.push(1);
    }
    var br = '';
    if (viewModel.OldPwd && viewModel.NewPwd && viewModel.ConfirmPwd && (viewModel.OldPwd.length < 6 || viewModel.NewPwd.length < 6 || viewModel.ConfirmPwd.length < 6)) {
        br += 'Password must great than 6 character<br/>';
        arrayErrors.push(1);
    }
    if (viewModel.OldPwd && viewModel.NewPwd && viewModel.OldPwd === viewModel.NewPwd) {
        br += 'Old Password and New Pass must different<br/>';
        arrayErrors.push(1);
    }
    if (viewModel.NewPwd !== viewModel.ConfirmPwd) {
        br += 'New Password and Confirm Password must match';
        arrayErrors.push(1);
    }
    if (br)
        MessageShowHide('alert-warning-top', br, 7000);
    if (arrayErrors.length > 0)
        return false;
    return true;
}
function updatePwd() {
    $('#btnUpdatePwd').prop('disabled', 'disabled');
    var viewModel = {
        OldPwd: $("#old-pwd").val(),
        ObjectId: getThisCookie(userObjectIdCookie),
        NewPwd: $("#new-pwd").val(),
        ConfirmPwd: $("#confirm-pwd").val()
    };
    if (!isValidUpdatePwd(viewModel)) {
        $('#btnUpdatePwd').removeProp('disabled');
        return;
    }
    var url = '/User/UpdatePassword';
    $.post(url, viewModel).then(updatePwdSuccess, updatePwdError);
    function updatePwdSuccess(response) {
        if (response.Success) {
            $('.alert-success').text('You have been change password successfully');
            $('.alert-success').show();
            setTimeout(function () {
                window.location.href = '/user/userprofile';
            }, 5000);
        }
        else {
            alert(response.Msg);
            $('#btnUpdatePwd').removeProp('disabled');
        }
    };
    function updatePwdError(e) {
        alert('There are error occur, please contact administrator.');
        $('#btnUpdatePwd').removeProp('disabled');
    };
}

function savePermission() {
    $('.alert-warning').hide();
    $('#btnSavePermission').prop('disabled', 'disabled');
    var arr = [];
    var obj = {};
    $('.newRoleUsers').each(function () {
        obj = {};
        obj.Id = $(this).attr('uid');
        obj.ObjectId = $(this).attr('oid');
        obj.UserName = $(this).attr('uname');
        obj.AuthenticationEmail = $(this).attr('email');  // $($(this).children()[0]).text();
        var role = $(this).attr('role');
        obj.Role = role.toLowerCase() == 'null' ? null : role;
        obj.IsEventOrganizer = $(this).find("input:checkbox").is(":checked"); // ADD Duc-BTT 20180307 Event作成権限付与画面
        arr.push(obj);
    });
    if (arr.length == 0) {
        $('.modal').modal('hide');
        $('.alert-warning strong').html('There are no permission changes in the list');
        $('.alert-warning').show();
        setTimeout(function () { $('.alert-warning').hide(); }, 5000);
        $('#btnSavePermission').removeProp('disabled');
        return;
    }
    $.ajax({
        type: 'post',
        url: '/User/UpdatePermission',
        data: { users: arr },
        success: success,
        error: error
    });
    function success(response) {
        $('.modal').modal('hide');
        if (response.Success) {
            MessageShowHide('alert-success', mimasResource.getResourceByKey('Permission Save'), 5000);
            $('alert-success').show();
            setTimeout(function () { $('.alert-success').hide(); }, 5000);
            dt.ajax.reload();
        }
        else {
            MessageShowHide('alert-warning', response.Msg, 5000);
        }
        $('#btnSavePermission').removeProp('disabled');
        $('#modalConfirm tbody').html('');
    };
    function error(xhr, status, error) {
        MessageShowHide('alert-warning', 'There are error occur, please contact admin', 5000);
        $('#modalConfirm tbody').html('');
    };
}

function browserSupportFileAPI() {
    if (window.File && window.FileReader && window.FileList && window.Blob && window.FormData) {
        return true;
    }
    return false;
}
function uploadCSVFile(evt) {
    $(evt.target).prop('disabled', 'disabled');
    if (!browserSupportFileAPI()) {
        alert('The File APIs are not fully supported in this browser!');
        $(evt.target).removeProp('disabled');
        return;
    }

    var fileElement = document.getElementById("inputFile");
    if (fileElement.files.length == 0) {
        alert('Please select a file.');
        $(evt.target).removeProp('disabled');
        return;
    }

    var file = fileElement.files[0];
    var extension = file.name.split(".");
    if (extension.length < 2 || extension.indexOf("csv") == -1 || extension[extension.length - 1].toLowerCase() != "csv") {
        alert('The import has failed due to wrong file format.');
        $(evt.target).removeProp('disabled');
        return;
    }

    var reader = new FileReader();
    reader.readAsText(file);
    reader.onload = function (event) {
        var csvData = event.target.result.trim();
        if (!csvData) {
            alert('The import file has failed due to wrong content format.');
            $(evt.target).removeProp('disabled');
            return;
        }

        var lines = csvData.split("\n");
        if (lines.length < 2) {
            alert('The import file has failed due to wrong content format.');
            $(evt.target).removeProp('disabled');
            return;
        }

        var errors = [];
        var hasErrorHead = false;
        var header = lines[0].split(",");
        if (header.length != 4)
            hasErrorHead = true;

        var email = header[0].trim();
        var password = header[1].trim();
        var displayName = header[2].trim();
        var department = header[3].trim();

        if (!email || !password || !displayName || !department)
            hasErrorHead = true;
        if (email.toLowerCase() != "email" || password.toLowerCase() != "password" || department.toLowerCase() != "department")
            hasErrorHead = true;
        if (displayName.toLowerCase() == "display name" || displayName.toLowerCase() == "displayname") {
        }
        else
            hasErrorHead = true;
        if (hasErrorHead)
            errors.push('The import file has wrong header format.');

        var linesValid = [];
        var mailformat = /^\w+([\.-]?\w+)*@\w+([\.-]?\w+)*(\.\w{2,3})+$/;
        lines.shift();
        var noErrorLine = true;
        var countEmpty = 0;
        for (var i in lines) {
            noErrorLine = true;
            var line = lines[i].split(",");
            if (line.length != 4) {
                noErrorLine = false;
                errors.push('The import has failed due to wrong line format, at line ' + (parseInt(i) + 1) + '.');
            }

            email = line[0] ? line[0].trim() : '';
            password = line[1] ? line[1].trim() : '';
            displayName = line[2] ? line[2].trim() : '';
            department = line[3] ? line[3].trim() : '';
            var columns = '';
            if (!email)
                columns = "email";
            if (!password)
                columns += (columns != '' ? ', ' : '') + "password";
            if (!displayName)
                columns += (columns != '' ? ', ' : '') + "display name";
            if (!department)
                columns += (columns != '' ? ', ' : '') + "department";

            if (columns) {
                noErrorLine = false;
                errors.push("You can't leave " + columns + ' blank, at line ' + (parseInt(i) + 1) + '.');
            }

            if (!email && !password && !displayName && !department) {
                countEmpty++;
            }

            if (email && !email.match(mailformat)) {
                noErrorLine = false;
                errors.push('You have entered an invalid email address, at line ' + (parseInt(i) + 1) + '.');
            }
            $('#tblFeature tbody').empty();

            if (password && password.length < 6) {
                noErrorLine = false;
                errors.push('Password must be at least 6 characters long, at line ' + (parseInt(i) + 1) + '.');
            }

            if (!noErrorLine) {
                continue;
            }

            linesValid.push(line);
        }

        if (countEmpty == lines.length) {
            errors = [];
            errors.push('The import has failed due to wrong content format.');
        }

        if (linesValid.length == 0) {
            $(evt.target).removeProp('disabled');
            alert(errors.join('\n'));
            return;
        }

        var url = '/User/ImportCSVAndRetrieve';
        $.ajax({
            url: url,
            type: 'POST',
            //enctype: 'multipart/form-data',
            //contentType: false,
            //processData: false,
            data: { lines: linesValid, rowsPerPage: $('#data-table_length select').val() },
            success: function (data) {
                var msgError = '';
                if (data.Success) {
                    paging(1, data);
                    if (data.Errors.trim())
                        msgError = data.Errors.trim();
                }
                else {
                    if (data.StepFail == 1) {
                        paging(1, data);
                        msgError = data.Msg1;
                    }
                    else if (data.StepFail == 2) {
                        msgError = data.Errors + '\n' + data.Msg2;
                    }
                    else if (data.StepFail == 12)
                        msgError = data.Msg1 + '\n' + data.Msg2;
                }
                if (msgError)
                    alert(errors.join('\n') + '\n' + msgError);
                else if (errors.length > 0)
                    alert(errors.join('\n'));

                $(evt.target).removeProp('disabled');
                $("#inputFile").val(null);
            },
            error: function (xhr, textStatus, errorThrown) {
                alert(errors.join('\n') + '\n\nThere are error occur, please contact administrator');
                $(evt.target).removeProp('disabled');
            }
        });
    };

    reader.onerror = function () {
        alert('The import has failed due to wrong file format.');
        $(evt.target).removeProp('disabled');
    };
}

function downloadCSVFile() {
    window.location.href = '/Templates/CSV File.csv';
}

var deactives = [];
function paging(page, data) {
    $('#tblFeature tbody').empty();
    var tr = '';
    var obj = {};
    for (var i = 0; i < data.Result.length; i++) {
        obj = data.Result[i];
        obj.DisplayName = obj.DisplayName == null ? '' : obj.DisplayName;
        obj.Department = obj.Department == null ? '' : obj.Department;
        var status = '';
        var btnClass = '';
        if (deactives.indexOf(obj.Id) != -1) {
            status = 'status="Inactive" value="Inactive"';
            btnClass = 'btn-info';
        }
        else {
            status = 'status="Active" value="Active"';
            btnClass = 'btn-primary';
        }
        tr = '<tr><td>' + obj.AuthenticationEmail + '</td><td>' + obj.DisplayName + '</td><td>' + obj.Department + '</td><td><input type="button" name="" class="btnActiveOrInactive btnActive btn ' + btnClass + ' btn-sm" uid="' + obj.Id + '" onclick="activeOrInactive(this)" ' + status + ' /></td></tr>';
        $('#tblFeature tbody').append(tr);
    }
    var rowsPerPage = parseInt($('#data-table_length select').val());
    console.log(rowsPerPage);
    var totalPage = parseInt(data.Total / rowsPerPage);
    var sodu = data.Total % rowsPerPage;
    totalPage = sodu > 0 ? (totalPage + 1) : totalPage;

    $('#paging ul').empty();
    var active = '';
    for (var i = 1; i <= totalPage; i++) {
        if (page == i)
            active = 'active';
        else
            active = '';
        $('#paging ul').append('<li><a href="javascript:" class="' + active + '" onclick="getImport(' + i + ')">' + i + '</a></li>');
    }
}

function activeOrInactive(t) {
    var status = $(t).attr('status');
    var indexArr = deactives.indexOf(parseInt($(t).attr('uid')));
    if (status == 'Active') {
        $(t).attr('status', 'Inactive');
        $(t).val('Inactive');
        $(t).removeClass('btn-primary');
        $(t).addClass('btn-info');
        if (indexArr == -1)
            deactives.push(parseInt($(t).attr('uid')));
    }
    else {
        $(t).attr('status', 'Active');
        $(t).val('Active');
        $(t).addClass('btn-primary');
        $(t).removeClass('btn-info');
        if (indexArr != -1) {
            deactives.splice(indexArr, 1);
        }
    }
}

function updateAndSendMail(t) {
    $(t).prop('disabled', 'disabled');
    $('.btnActiveOrInactive').prop('disabled', 'disabled');

    $.ajax({
        url: '/User/UpdateAndSendMail',
        type: 'POST',
        data: { ids: deactives },
        success: function (data) {
            if (data.Success) {
                if (data.Failed.length > 0) {
                    var mailFails = 'These mail send was failed : \n';
                    for (var i = 0; i < data.Failed.length; i++) {
                        mailFails += data.Failed[i] + '\n';
                    }
                    alert(mailFails);
                }
                else {
                    alert('Send mail successfully');
                }
                getImport(1);
            }
            else {
                alert('There are error occur, please contact administrator');
            }
            $(t).removeProp('disabled');
            $('.btnActiveOrInactive').removeProp('disabled');
        },
        error: function (xhr, textStatus, errorThrown) {
            alert('There are error occur, please contact administrator');
        }
    });
}
