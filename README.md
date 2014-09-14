Anonymousphp
============

When building apps that involve scraping, downloading data and automation, staying completely anonymous can be a huge concern for developers. Although there are many different proxy checkers out there, most of them all seem to deliver slightly different and unreliable results.

This guide will walk you through the three steps and provide clean PHP code to detect exactly how anonymous a specific proxy is.

Elite (High-Anonymous)
Your proxy is completely undetectable and your real IP will remain hidden. The server you connect to will have no idea you’re using a proxy. These are the best proxies you will find and the level of anonymity and quality is unprecedented.

Anonymous
Although your proxy IP is still hidden while connected to an anonymous proxy, some servers and proxy detection scripts will be able to detect that you’re using a proxy. Although these proxies are still useful for whitehat practices and data mining, your original IP still has a slight chance of exposure.

Transparent
Your original IP will be exposed and everyone will know you’re using a proxy. It is extremely risky and highly recommended to avoid using transparent proxies while trying to remain anonymous.

Step 1: Create a Proxy Gateway
The first step is to set up a gateway on your server that will emulate what any other server will use to determine if you’re using a proxy using the $_SERVER superglobal. Make sure this PHP file is accessible through a public URL (e.g. http://yourdomain.com/gateway.php)

Since $_SERVER outputs as an array, you’ll need to do some formatting. Here is an example of how I formatted the output in gateway.php as a string to easily extract the data for the proxy anonymity tester:

$output=';

foreach ($_SERVER as $key => $value) {
    if (!empty($value)) {
        $output .= $key . '--' . $value . '---';
    }
}

$output=substr($output, 0, -3);

Step 2: Connect to the Server Gateway and Retrieve Results
Once your gateway is set up, you’re ready to connect to it with your proxy and retrieve the $_SERVER output which will reveal how anonymous the proxy is. Below is some simple PHP code using cURL to access your gateway URL. This simple cURL script will detect if your proxy’s protocol is HTTP, SOCKS4, SOCKS5 or SOCKS4/5, so no need to determine that beforehand.

Note: Make sure the $url variable is set to your gateway URL and the $proxy variable is set to the proxy you’d like to test (in IP:PORT format).

public function gatewayResults($url, $proxy) {
    $types=array(
        'http',
        'socks4',
        'socks5'
    );

    $url=curl_init($url);

    curl_setopt($url, CURLOPT_PROXY, $proxy);

    foreach ($types as $type) {
        switch ($type) {
            case 'http':
                curl_setopt($url, CURLOPT_PROXYTYPE, CURLPROXY_HTTP);
                break;
            case 'socks4':
                curl_setopt($url, CURLOPT_PROXYTYPE, CURLPROXY_SOCKS4);
                break;
            case 'socks5':
                curl_setopt($url, CURLOPT_PROXYTYPE, CURLPROXY_SOCKS5);
                break;
        }

        curl_setopt($url, CURLOPT_TIMEOUT, 10);
        curl_setopt($url, CURLOPT_RETURNTRANSFER, 1);

        $resultsQuery=explode('---', curl_exec($url));

        if (!empty($resultsQuery)) {
            break;
        }
    }

    $results=array();

    foreach ($resultsQuery as $result) {
        if (!empty($result)) {
            $split=explode('--', $result);

            if (!empty($split[1])) {
                $results[$split[0]]=$split[1];
            }
        }
    }

    curl_close($url);
    unset($url);

    return $results;
} 

Step 3: Check Proxy Anonymity by Using the Gateway Results

After you have the returned server data from the above gatewayResults function, simply pass it to the function below and it will return the proxy anonymity level.

public function checkAnonymity($server=array()) {
    $realIp=$_SERVER['REMOTE_ADDR];

    $level=transparent';

    if (!in_array($realIp, $server)) {
        $level=anonymous';

        $proxyDetection=array(
            'HTTP_X_REAL_IP',
            'HTTP_X_FORWARDED_FOR',
            'HTTP_X_PROXY_ID',
            'HTTP_VIA',
            'HTTP_X_FORWARDED_FOR',
            'HTTP_FORWARDED_FOR',
            'HTTP_X_FORWARDED',
            'HTTP_FORWARDED',
            'HTTP_CLIENT_IP',
            'HTTP_FORWARDED_FOR_IP',
            'VIA',
            'X_FORWARDED_FOR',
            'FORWARDED_FOR',
            'X_FORWARDED FORWARDED',
            'CLIENT_IP',
            'FORWARDED_FOR_IP',
            'HTTP_PROXY_CONNECTION',
            'HTTP_XROXY_CONNECTION'
        );

        if (!array_intersect(array_keys($server), $proxyDetection)) {
            $level=elite';
        }
    }

    return $level;
} 

die($output); 
