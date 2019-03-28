# wp-list-table
wp-list-table
 
# init.php
```
<?php
 /*
    Plugin Name: Rating Provider
    Plugin URI: http://www.xyz.com/
    Description: Plugin for displaying Rating of Provider.
    Author: xyz
    Version: 4.0
    Author URI: http://www.xyz.com/
    */	
// function to create the DB / Options / Defaults					
function rating_options_install() {

    global $wpdb;

    $table_name = $wpdb->prefix . "rating";
    $charset_collate = $wpdb->get_charset_collate();
    $sql = "CREATE TABLE $table_name (
            `id` int  NOT NULL AUTO_INCREMENT,
            `post_id` varchar(255) CHARACTER SET utf8 NOT NULL,
			`created_date` varchar(255) CHARACTER SET utf8 NOT NULL,
			`provider` varchar(255) CHARACTER SET utf8 NOT NULL,
			`overall_rating` varchar(255) CHARACTER SET utf8 NOT NULL,
			`service` varchar(255) CHARACTER SET utf8 NOT NULL,
			`price` varchar(255) CHARACTER SET utf8 NOT NULL,
			`speed` varchar(255) CHARACTER SET utf8 NOT NULL,
			`email` varchar(255) CHARACTER SET utf8 NOT NULL,
			`fname` varchar(255) CHARACTER SET utf8 NOT NULL,
			`lname` varchar(255) CHARACTER SET utf8 NOT NULL,
			`zip` varchar(255) CHARACTER SET utf8 NOT NULL,
			`comments` varchar(255) CHARACTER SET utf8 NOT NULL,
			`like` varchar(255) CHARACTER SET utf8 NOT NULL,
			`dislike` varchar(255) CHARACTER SET utf8 NOT NULL,
            PRIMARY KEY (`id`)
          ) $charset_collate; ";
		  
	


require_once( ABSPATH . 'wp-admin/includes/upgrade.php' );
    dbDelta($sql);
	
}

// run the install scripts upon plugin activation
register_activation_hook(__FILE__, 'rating_options_install');

//menu items
add_action('admin_menu','rating_modifymenu');
function rating_modifymenu() {
	
	//this is the main item for the menu
	add_menu_page('Rating Provider', //page title
	'Rating Provider', //menu title
	'manage_options', //capabilities
	'rating_list', //menu slug
	'rating_provider_list', //function
	'dashicons-star-half'
	);	
	
}
define('ROOTDIR', plugin_dir_path(__FILE__));
require_once(ROOTDIR . 'rating-list.php');
require_once(ROOTDIR . 'rating-form.php');
require_once(ROOTDIR . 'table-list-class.php');
```
 
# table-list-class.php
```
<?php if(!class_exists('WP_List_Table')){
    require_once( ABSPATH . 'wp-admin/includes/class-wp-list-table.php' );
    
}
class TT_Example_List_Table extends WP_List_Table {

 var $example_data;
 

        

	function __construct(){
	    
	    global $wpdb;
        $table_name1 = $wpdb->prefix .'rating';
		$sql1 = "select * from $table_name1";
		$rowss = $wpdb->get_results($sql1);
		//print_r($rowss);
			foreach ($rowss as $rowqq) { 
			    $this->example_data[]=json_decode(json_encode($rowqq), TRUE);
			}
        global $status, $page;
                
        //Set parent defaults
        parent::__construct( array(
            'singular'  => 'movie',     //singular name of the listed records
            'plural'    => 'movies',    //plural name of the listed records
            'ajax'      => false        //does this table support ajax?
        ) );
        
    }


    function column_default($item, $column_name){
        switch($column_name){
            case 'provider':
            case 'overall_rating':
            case 'service':
            case 'price':
            case 'speed':
            case 'fname':
            case 'email':
            case 'comments':
            case 'like':
            case 'dislike':    
                return $item[$column_name];
            default:
                return print_r($item,true); //Show the whole array for troubleshooting purposes
        }
    }


    function column_provider($item){
        
        //Build row actions
        $actions = array(
            'edit'      => sprintf('<a href="?page=%s&action=%s&movie=%s">Edit</a>',$_REQUEST['page'],'edit',$item['id']),
            'delete'    => sprintf('<a href="?page=%s&action=%s&movie=%s">Delete</a>',$_REQUEST['page'],'delete',$item['id']),
        );
        
        //Return the title contents
        return sprintf('%1$s <span style="color:silver">(id:%2$s)</span>%3$s',
            /*$1%s*/ $item['provider'],
            /*$2%s*/ $item['id'],
            /*$3%s*/ $this->row_actions($actions)
        );
    }
    
    function column_overall_rating($item){
        
      
                                            $totalRating = 5;
                                            $starRatingSER=$item['overall_rating'];
                                            for ($i = 1; $i <= $totalRating; $i++) {
                                                 if($starRatingSER < $i ) {
                                                    if(round($starRatingSER) == $i){ 
                                                      echo '<img src="'.get_template_directory_uri().'/images/half.png">';
                                                    }else{ 
                                                       echo '<img src="'.get_template_directory_uri().'/images/empty.png">';
                                                    }
                                                 }else{ 
                                                   echo '<img src="'.get_template_directory_uri().'/images/full.png">';
                                               }
                                            }
        return sprintf('<br> <span style="color:silver">%1$s</span>',
            /*$1%s*/ $item['overall_rating']
        );
                                        
    }


    function column_cb($item){
        return sprintf(
            '<input type="checkbox" name="%1$s[]" value="%2$s" />',
            /*$1%s*/ $this->_args['singular'],  //Let's simply repurpose the table's singular label ("movie")
            /*$2%s*/ $item['id']                //The value of the checkbox should be the record's id
        );
    }


    function get_columns(){
        $columns = array(
            'cb'        => '<input type="checkbox" />', //Render a checkbox instead of text
            'provider'     => 'Provider',
            'overall_rating'    => 'Overall rating',
            'service'    => 'Service rating',
            'price'    => 'Price rating',
            'speed'    => 'Speed rating',
            'fname'  => 'User',
            'email'  => 'Email',
            'comments'  => 'Comments',
            'like'  => 'Like',
            'dislike'  => 'Dislike'
            
        );
        return $columns;
    }


    function get_sortable_columns() {
        $sortable_columns = array(
            'provider'     => array('provider',false),     //true means it's already sorted
            'overall_rating'    => array('overall_rating',false),
            'fname'  => array('fname',false)
        );
        return $sortable_columns;
    }

	function get_bulk_actions() {
        $actions = array(
            'delete'    => 'Delete'
        );
        return $actions;
    }
	function process_bulk_action() {
        
        //Detect when a bulk action is being triggered...
        if( 'delete'===$this->current_action() ) {
            wp_die('Items deleted (or they would be if we had items to delete)!');
        }
        
    }


    function prepare_items() {
        global $wpdb; //This is used only if making any database queries

        /**
         * First, lets decide how many records per page to show
         */
        $per_page = 15;
        $columns = $this->get_columns();
        $hidden = array();
        $sortable = $this->get_sortable_columns();
        $this->_column_headers = array($columns, $hidden, $sortable);
        $this->process_bulk_action();
        $data = $this->example_data;
        function usort_reorder($a,$b){
            $orderby = (!empty($_REQUEST['orderby'])) ? $_REQUEST['orderby'] : 'provider'; //If no sort, default to title
            $order = (!empty($_REQUEST['order'])) ? $_REQUEST['order'] : 'asc'; //If no order, default to asc
            $result = strcmp($a[$orderby], $b[$orderby]); //Determine sort order
            return ($order==='asc') ? $result : -$result; //Send final sort direction to usort
        }
        usort($data, 'usort_reorder');
        
       
        $current_page = $this->get_pagenum();
        
        $total_items = count($data);
        $data = array_slice($data,(($current_page-1)*$per_page),$per_page);
        $this->items = $data;
        
		$this->set_pagination_args( array(
            'total_items' => $total_items,                  //WE have to calculate the total number of items
            'per_page'    => $per_page,                     //WE have to determine how many items to show on a page
            'total_pages' => ceil($total_items/$per_page)   //WE have to calculate the total number of pages
        ) );
    }

}


?>
```

# rating-list.php
```
<?php
function rating_provider_list() {

        $testListTable = new TT_Example_List_Table();
        $testListTable->prepare_items();

?>
      <div class="wrap">

          <div id="icon-users" class="icon32"><br/></div>
          <h2>List Rating</h2>
         <form id="movies-filter" method="get">
              <!-- For plugins, we also need to ensure that the form posts back to our current page -->
              <input type="hidden" name="page" value="<?php echo $_REQUEST['page'] ?>" />
              <!-- Now we can render the completed list table -->
              <?php $testListTable->display() ?>
          </form>

      </div>
<? } ?>
```
