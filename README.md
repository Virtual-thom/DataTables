# DataTables

## oracle.ssp.class.php

Petite adaptation pour ma base oracle afin d'effectuer du serverSide Processing avec DataTables.

Originale pour MySQL :

[https://github.com/DataTables/DataTables/blob/master/examples/server_side/scripts/ssp.class.php](https://github.com/DataTables/DataTables/blob/master/examples/server_side/scripts/ssp.class.php)

[https://datatables.net/examples/data_sources/server_side](https://datatables.net/examples/data_sources/server_side)

## api exemple
```php
<?php
ini_set('max_execution_time', 300);
error_reporting(E_ALL);
ini_set("display_errors", 1);
ini_set('memory_limit', '512M');

# return content as json
header('Content-Type: application/json');

require( 'oracle.ssp.class.php' );
$fic_par = 'cnx_ora.ini';
if(!file_exists($fic_par)) {
		echo("Le fichier ". $fic_par ." est introuvable");
		die();
}
//Recuperation User/Mot de passe/Instance
$ini = parse_ini_file($fic_par,TRUE);
$sql_details = array(
	'user' => $ini['MONID']['User'],
	'pass' => $ini['MONID']['Mdp'],
	'dbInstance'   => $ini['MONID']['Instance']
);


// DB table to use
$table = 'MATABLE';

// Table's primary key
$primaryKey = 'MONINDEX';

$columns = array(
	array( 'db' => 'INDEX', 'dt' => 0 ),
	array( 'db' => 'TRTNAME',  'dt' => 1 ),
	array( 'db' => 'ENVNAME',   'dt' => 2 ),
	array( 'db' => 'FICTRT',     'dt' => 3 ),
	array(
		'db'        => 'DATE_DEB',
		'dt'        => 4/*,
		'formatter' => function( $d, $row ) {
			return date( 'jS M y', strtotime($d));
		}*/
	),
	array(
		'db'        => 'DATE_FIN',
		'dt'        => 5/*,
		'formatter' => function( $d, $row ) {
			return date( 'jS M y', strtotime($d));
		}*/
	),
	array( 'db' => 'INFO',     'dt' => 6 )
);


echo json_encode(
	SSP::simple( $_GET, $sql_details, $table, $primaryKey, $columns )
);
```

et les appels en js par ex 

```
  "processing": true,
  "serverSide": true,
  "ajax": "/api/dtServerSideProcessing.php",
 ```

```javascript
$(document).ready(function() {

var table = $('#example').DataTable({
  "dom": 'Bfrtip',
  "language": {"url": "vendor/DataTables-1.10.18/lang/French.json"},
  "processing": true,
  "serverSide": true,
  "ajax": "/api/dtServerSideProcessing.php",
  "order": [[ 4, "desc" ], [ 0, "desc"], [2, "desc"]],
  "lengthMenu": [ [10, 25, 50, -1], [10, 25, 50, "Tout"] ],
  "buttons": [
    'pageLength',
    'colvis',
    'copy',
    'excel',
    {
      extend : 'pdfHtml5',
      title : function() {
          return "USI Suivi Trt";
      },
      orientation : 'landscape',
      pageSize : 'LEGAL',
      text : 'PDF',
      titleAttr : 'PDF'
    },
    {
    extend : 'print',
    customize: function(win)
            {
                var last = null;
                var current = null;
                var bod = [];

                var css = '@page { size: landscape; }',
                    head = win.document.head || win.document.getElementsByTagName('head')[0],
                    style = win.document.createElement('style');
 
                style.type = 'text/css';
                style.media = 'print';
 
                if (style.styleSheet)
                {
                  style.styleSheet.cssText = css;
                }
                else
                {
                  style.appendChild(win.document.createTextNode(css));
                }
 
                head.appendChild(style);
      }
    },
    {
      text: 'Refresh',
      action: function ( e, dt, node, config ) {
          dt.ajax.reload();
      }
    }
  ],
  "initComplete": function() {
    var api = this.api();

    // Apply the search
    api.columns().every(function() {
      var that = this;

      $('input', this.footer()).on('keyup change', function() {
        if (that.search() !== this.value) {
          that
            .search(this.value)
            .draw();
        }
      });
    });
  },
  "colReorder": true
});


// Setup - add a text input to each footer cell
$('#example tfoot th').each( function () {
  var title = $(this).text();
  $(this).html( '<input type="text" placeholder="Filtre '+title+'" />' );
} );

}); // end ready
```
