You can disable some or all of the default endpoints provided by the WordPress REST API. You should be very careful about doing so.  The default routes will be counted on by many third party tools. In the future, it is likely that the WordPress admin will begin to utilize the REST API, and removing the default routes will break the admin.

You can remove the registration of the default routes, wholesale, by removing the hook where those endpoints were are added, like this:

`remove_action( 'rest_api_init', 'create_initial_rest_routes', 0 );`

You can also disable 1 or more endpoint ussing the `rest_endpoints` filter. If, for example, you wanted to remove all default endpoints for media, you would need to search the URLs for the endpoints in the wp/v2 namespace that include "media" somewhere in their URL, like this:

```
add_filter( 'rest_endpoints', function( $endpoints ) {
  foreach( $endpoints as $endpoint => $args ) {
     if ( isset( $args[ 'namespace' ] ) && 'wp/v2' == $args[ 'namespace' ] ) {
        $parsed_route = explode( '/', $endpoint );
        if ( in_array( 'media', $parsed_route ) ) {
           unset( $endpoints[ $endpoint ] );
        }
  } }
  return $endpoints;
});
```

Or you can disable just one set of endpoints. For example, to disable users endpoints:

```
add_filter( 'rest_endpoints', function( $endpoints ){
    if ( isset( $endpoints['/wp/v2/users'] ) ) {
        unset( $endpoints['/wp/v2/users'] );
    }
    if ( isset( $endpoints['/wp/v2/users/(?P<id>[\d]+)'] ) ) {
        unset( $endpoints['/wp/v2/users/(?P<id>[\d]+)'] );
    }
    return $endpoints;
});
```


### Make API Require Login
You can also use the `rest_authentication_errors` filter to prevent any requests to the REST API to non-logged in users.

```
add_filter( 'rest_authentication_errors', function( $result ) {
	if ( ! empty( $result ) ) {
		return $result;
	}
	if ( ! is_user_logged_in() ) {
		return new WP_Error( 'restx_logged_out', 'Sorry, you must be logged in to make a request.', array( 'status' => 401 ) );
	}
	return $result;
});

```