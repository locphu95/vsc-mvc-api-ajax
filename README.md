# vsc-mvc-api-ajax

@{
    ViewData["Title"] = "Home Page";
}
 <select name="sodong" class="btn btn-default dropdown-toggle">
                              <option>5</option>
  <option selected="selected">7</option>
  <option>10</option>
  <option selected="selected">15</option>
    </select>
<table class="table">
    <thead>
      <tr>
        <th>Ma sinh vien</th>
        <th>Ten sinh vien</th>
        <th>ma khoa</th>
      </tr>
    </thead>
    <tbody id="tbody1">
    
    </tbody>

  </table>
    <div id="paging" class="dataTables_paginate paging_simple_numbers">
      <ul class="pagination"></ul>
    </div>

    <select id="data-table_length"  class="form-control input-sm">
                                            <option value="5">5</option>
                                            <option value="10">10</option>
                                            <option value="15">15</option>
                                        </select>
    <table id="tblFeature" class="table">
        <thead>
      <tr>
        <th>Ma sinh vien</th>
        <th>Ten sinh vien</th>
        <th>ma khoa</th>
      </tr>
    </thead> 
    <tbody id="tbody">
    
    </tbody>

    </table>

@section scripts {

<script>
$(document).ready(function () {
  loadsv(1)
});
function loadsv(page){
        $.ajax({
            type: 'GET',
            url: '/api/Sinhvien/',
            dataType: "json",
            success: function (data) {
                //getData(1,data);
                paging(1, data);
                console.log(data);
            },
            error: function (request, status, error) {
                alert(error);
            }
        });
}
getImport(1);
    function getImport(page) {
        $.ajax({
            url: '/User/GetAllByStatus',
            type: 'POST',
            data: { status: 4, page: page, rowsPerPage: $('#data-table_length select').val() },
            success: function (data) {
                if (data.Success) {
                    paging(1, data);
                }
            },
            error: function (xhr, textStatus, errorThrown) {
            }
        });
    }
function paging(page, data) {
    console.log("so luong du lieu"+ " "+data.length);
    $('#tblFeature tbody').empty();
    var tr = '';
    var obj = {};
    for (var i = 0; i < data.length; i++) {
        obj = data[i];
        obj.DisplayName = obj.name_sv == null ? '' : obj.name_sv;
        obj.Department = obj.id_khoa == null ? '' : obj.id_khoa;
        var status = '';
        tr = '<tr><td>' + obj.id_sv + '</td><td>' + obj.name_sv + '</td><td>' + obj.id_khoa + '</td></tr>';
        $('#tblFeature tbody').append(tr);
    }
    var sodong = parseInt($('#data-table_length select').val());
    console.log("sodong"+ sodong);
   // var sotrang =(s/slt);
   // var sodu=s%slt ;
    //sotrang = sodu > 0 ? (sotrang + 1) : sotrang;

    $('#paging ul').empty();
    var active = '';
    for (var i = 1; i <= 9; i++) {
        if (page == i)
            active = 'active';
        else
            active = '';
        $('#paging ul').append('<li><a href="javascript:" class="' + active + '" onclick="getImport(' + i + ')">' + i + '</a></li>');
    }
}
// function getData(page,data) { 
//     console.log(data.length);
  
    
// $( "select" )
//   .change(function () {
//     var str = "";
//     $( "select option:selected" ).each(function() {
//       str += $( this ).text() + " ";
//     });
//     console.log("chon sl xem" + str);
//     var x = data.length;
//     $("#tbody").html("");
//     for(var i =0;i<=data.length;i++)
//     if(i<parseInt(str))
//       { console.log(data[i]);
//          $("#tbody1").append(loadcbnv(data[i]));
//       }
//     getsotrang(x,str);
//   })
// }
// function getsotrang(s,slt){
//     var sotrang =(s/slt);
//     var sodu=s%slt ;
//     sotrang = sodu > 0 ? (sotrang + 1) : sotrang;
//     console.log("so du="+sodu);
//     console.log("so trang="+sotrang);
//   $('#paging ul').empty();
//     var active = '';
//     for (var i = 1; i <=sotrang; i++) {
//         if (slt == i)
//             active = 'active';
//         else
//             active = '';
//         $('#paging ul').append('<li><a href="javascript:" class="' + active + '" onclick="getImport(' + i + ')">' + i + '</a></li>');
//     }
// }

function loadcbnv(item) {
    detailsHtml ='<tr>';
    detailsHtml += '<td>'+item.id_sv+'</td>'+
    '<td>'+item.name_sv+'</td>'+
    '<td>'+item.id_khoa +'</td>'+
    '<td>'
    +'<button type="button" class=" btn btn-info fa fa-edit showcn" onclick="suaCbnv(\''+ item.maCBNV +'\')"><span class="tooltiptext">Sửa</span></button>|<button type="button" class="btn btn-info fa fa-file-text-o showcn" onclick="chitietCbnv(\'' + item.maCBNV + '\')"><span class="tooltiptext">Chi tiết</span></button>|<button type="button" class="btn btn-info fa fa-trash-o showcn" onclick="xoaCbnv(\'' + item.maCBNV + '\')"><span class="tooltiptext">Xóa </span></button>'
    detailsHtml +='</tr>';  
    return detailsHtml;
}
</script>
}
