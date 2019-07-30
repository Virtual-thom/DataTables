# exemple d'utilisation DataTable Server-Side Processing php / Oracle

`index.php`
```php
<?php
// DB table to use
$table = 'TOP100_STATS_TRT';
// Table's primary key
$primaryKey = 'VTENVNAME';


$columns = array(
	array( 'db' => 'VTENVNAME',       'dt' => 0  , "priority" => 1   ),
	array( 'db' => 'VTAPPLNAME',      'dt' => 1  , "priority" => 2   ),
	array( 'db' => 'VTJOBNAME',       'dt' => 2  , "priority" => 3   ),
	array( 'db' => 'VTSTATUS',        'dt' => 3  , "priority" => 4   , 'formatter' => 'function($v){$status[3] = "OK" ; $status[4] = "ERR" ; return $status[$v];}' ), 
	array( 'db' => 'VTBEGIN',         'dt' => 4  , "priority" => 255 ),
	array( 'db' => 'VTEND',           'dt' => 5  , "priority" => 5   ),
	array( 'db' => 'VTERRMESS',       'dt' => 6  , "priority" => 255 ),
	array( 'db' => 'VTEXPDATEVALUE',  'dt' => 7  , "priority" => 255 ),
	array( 'db' => 'VTDOMAINE',       'dt' => 8  , "priority" => 5   ),
	array( 'db' => 'VTFAMILY',        'dt' => 9  , "priority" => 255 ),
	array( 'db' => 'VTMACHINE',       'dt' => 10 , "priority" => 255 ),
	array( 'db' => 'VTQUEUE',         'dt' => 11 , "priority" => 255 ),
	array( 'db' => 'VTUSER',          'dt' => 12 , "priority" => 255 ),
	array( 'db' => 'VTCOMMENT',       'dt' => 13 , "priority" => 255 ),
	array( 'db' => 'VTELAPSE_SEC',    'dt' => 14 , "priority" => 255 ),
	array( 'db' => 'VTSCRIPT',        'dt' => 15 , "priority" => 6   ),
	array( 'db' => 'VTDATENAME',      'dt' => 16 , "priority" => 255 ),
	array( 'db' => 'VTEXETYPECODE',   'dt' => 17 , "priority" => 7  , 'formatter' => 'function($v){$modeexec = ["", "EXEC", "SIMU", "TEST", "STOP" ]; return $modeexec[$v] ;}' )
);

?>
<!DOCTYPE html>
<html lang="fr">

<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
  <title>TOP100 STATS</title>
  <link rel="stylesheet" type="text/css" href="/api/vendor/datatables.min.css"/>
  <link rel="stylesheet" type="text/css" href="index.css"/>
</head>

<body>
    <table id="example" class="display" style="width:100%">
        <thead>
            <tr>
            <?php 
            foreach($columns as $k => $column){
              echo "<th data-priority='".$column['priority']."'>".$column['db']."</th>" ;
            }
            ?>
            </tr>
        </thead>
        <tfoot>
            <tr>
            <?php 
            foreach($columns as $k => $column){
              echo "<th>".$column['db']."</th>" ;
            }
            ?>
            </tr>
        </tfoot>
    </table>
<script src="/api/vendor/jquery-3.3.1.min.js" type="text/javascript"></script>
<script src="/api/vendor/datatables.min.js" type="text/javascript"></script>
<script>
window.myColumns = <?php echo json_encode($columns); ?> ;
window.myTable = <?php echo json_encode($table); ?> ;
window.myPrimaryKey = <?php echo json_encode($primaryKey); ?> ;
</script>
<script src="index.js" type="text/javascript"></script>
</body>

</html>
```

`index.js`
```js
$(document).ready(function () {

  function throttle(callback, delay) {
    var last;
    var timer;
    return function () {
      var context = this;
      var now = +new Date();
      var args = arguments;
      if (last && now < last + delay) {
        // le délai n'est pas écoulé on reset le timer
        clearTimeout(timer);
        timer = setTimeout(function () {
          last = now;
          callback.apply(context, args);
        }, delay);
      } else {
        last = now;
        callback.apply(context, args);
      }
    };
  }

  var table = $('#example').DataTable({
    "dom": 'Bfrtip',
    "language": { "url": "/api/vendor/DataTables-1.10.18/lang/French.json" },
    "processing": true,
    "serverSide": true,
    "ajax": {
      "url": "/api/dtServerSideProcessing.php",
      "data": function (d) {
        let objTable = {}
        objTable["base"] = "POUTI";
        if (window.myColumns) {
          objTable["myColumns"] = window.myColumns
        }
        if (window.myTable) {
          objTable["myTable"] = window.myTable
        }
        if (window.myPrimaryKey) {
          objTable["myPrimaryKey"] = window.myPrimaryKey
        }
        return $.extend({}, d, objTable)
      }
    },
    "order": [
      [4, "desc"]
      /*
      [0,"asc"]
      ,[1,"asc"]
      ,[2,"asc"]
      ,[4,"desc"]
      ,[5,"desc"]
      ,[8,"asc"]
      */
    ],
    "ordering": true,
    "buttons": [
      'copy',
      'excel',
      'pageLength'
    ],
    'responsive': true,
    "initComplete": function () {
      var api = this.api();

      // Apply the search
      api.columns().every(function () {
        var that = this;

        $('input', this.footer()).on('keyup change', throttle(function () {
          if (that.search() !== this.value) {
            that
              .search(this.value)
              .draw();
          }
        }, 3000));
      });
    },
    "colReorder": true,
    "lengthMenu": [[10, 25, 50, -1], [10, 25, 50, "All"]]
  });


  // Setup - add a text input to each footer cell
  $('#example tfoot th').each(function () {
    var title = $(this).text();
    $(this).html('<input type="text" placeholder="Search ' + title + '" />');
  });

}); // end ready
```

`dtServerSideProcessing.php`
```php
<?php
ini_set('max_execution_time', 300);
error_reporting(E_ALL);
ini_set("display_errors", 1);
ini_set('memory_limit', '512M');

# return content as json
header('Content-Type: application/json');

require( '../oracle.ssp.class.php' );


$base = "maBase";
if(isset($_REQUEST["base"]) && $_REQUEST["base"] != ""){
	$base = $_REQUEST["base"];
}

$fic_par = '../cnx_ora.ini';
if(!file_exists($fic_par)) {
		echo("Le fichier ". $fic_par ." est introuvable");
		die();
}
//Recuperation User/Mot de passe/Instance
$ini = parse_ini_file($fic_par,TRUE);
$sql_details = array(
	'user' => $ini[$base]['User'],
	'pass' => $ini[$base]['Mdp'],
	'dbInstance'   => $ini[$base]['Instance']
);

if(isset($_REQUEST['myTable'])){
	$table = $_REQUEST['myTable'] ;
}
if(isset($_REQUEST['myPrimaryKey'])){
	$primaryKey = $_REQUEST['myPrimaryKey'] ;
}
if(isset($_REQUEST['myColumns'])){
	$columns = $_REQUEST['myColumns'] ;
}

echo json_encode(
	SSP::simple( $_GET, $sql_details, $table, $primaryKey, $columns )
);
```
