@{
    ViewData["Title"] = "Home Page";
}
 <select name="sodong" class="btn btn-default dropdown-toggle" onchange="loadsv(1)">
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
    <tbody id="tbody">
    
    </tbody>

  </table>
    <div id="paging" class="dataTables_paginate paging_simple_numbers">
      <ul class="pagination"></ul>
    </div>


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
                getData(1,data);
                //paging(1, data);
                console.log(data);
            },
            error: function (request, status, error) {
                alert(error);
            }
        });
}
// function paging(page, data) {
//     console.log("so luong du lieu"+ " "+data.length);
//     $('#tblFeature tbody').empty();
//     var tr = '';
//     var obj = {};
//     for (var i = 0; i < data.length; i++) {
//         obj = data[i];
//         obj.DisplayName = obj.name_sv == null ? '' : obj.name_sv;
//         obj.Department = obj.id_khoa == null ? '' : obj.id_khoa;
//         var status = '';
//         tr = '<tr><td>' + obj.id_sv + '</td><td>' + obj.name_sv + '</td><td>' + obj.id_khoa + '</td></tr>';
//         $('#tblFeature tbody').append(tr);
//     }
//     var sodong = parseInt($('#data-table_length select').val());
//     console.log("sodong"+ sodong);
//    // var sotrang =(s/slt);
//    // var sodu=s%slt ;
//     //sotrang = sodu > 0 ? (sotrang + 1) : sotrang;

//     $('#paging ul').empty();
//     var active = '';
//     for (var i = 1; i <= 9; i++) {
//         if (page == i)
//             active = 'active';
//         else
//             active = '';
//         $('#paging ul').append('<li><a href="javascript:" class="' + active + '" onclick="getImport(' + i + ')">' + i + '</a></li>');
//     }
// }
 function getData(page,data) { 
     console.log(data.length);
 $( "select" )
   .change(function () {
     var str = "";
     $( "select option:selected" ).each(function() {
       str += $( this ).text() + " ";
     });
     var x = data.length;
     console.log("chon sl xem" + str);
     $("#tbody").html("");
    for(var i =0;i<=data.length;i++)
   if(i<parseInt(str))
     { console.log(data[i]);
         $("#tbody").append(loadcbnv(data[i]));
     }
     getsotrang(x,str);
   })
 }
 function getsotrang(s,slt){
     var sotrang =(s/slt);
     var sodu=s%slt ;
     sotrang = sodu > 0 ? (sotrang + 1) : sotrang;
     console.log("so du="+sodu);
     console.log("so trang="+sotrang);
   $('#paging ul').empty();
    $("#paging ul").append("<li id='next-page'><a href='javascript:void(0)' aria-label=Previous><span aria-hidden=true><<</span></a></li>"); 

     var active = '';
     for (var i = 1; i <=sotrang; i++) {
        {
         if (slt == i)
             active = 'active';
         else
             active = '';
         $('#paging ul').append('<li><a href="javascript:" class="' + active + '" onclick="loadsv(' + i + ')">' + i + '</a></li>');
        }
    }$("#paging ul").append("<li id='next-page'><a href='javascript:void(0)' aria-label=Next><span aria-hidden=true>&raquo;</span></a></li>"); 
    $(".pagination li.current-page").on("click", function() {
  // Check if page number that was clicked on is the current page that is being displayed
  if ($(this).hasClass('active')) {
    return false; // Return false (i.e., nothing to do, since user clicked on the page number that is already being displayed)
  } else {
    var currentPage = $(this).index(); // Get the current page number
    $(".pagination li").removeClass('active'); // Remove the 'active' class status from the page that is currently being displayed
    $(this).addClass('active'); // Add the 'active' class status to the page that was clicked on
    $("#page .list-group").hide(); // Hide all items in loop, this case, all the list groups
    var grandTotal = limitPerPage * currentPage; // Get the total number of items up to the page number that was clicked on

    // Loop through total items, selecting a new set of items based on page number
    for (var i = grandTotal - limitPerPage; i < grandTotal; i++) {
      $("#page .list-group:eq(" + i + ")").show(); // Show items from the new page that was selected
    }
  }

});
}
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
