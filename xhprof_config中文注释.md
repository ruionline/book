<?php
$_xhprof = array();

// Change these:
$_xhprof['dbtype'] = 'mysql'; // Only relevant for PDO
$_xhprof['dbhost'] = '';//MYSQL服务器IP
$_xhprof['dbuser'] = '';//用户名
$_xhprof['dbpass'] = '';//用户密码
$_xhprof['dbname'] = '';//数据库名
$_xhprof['dbadapter'] = 'Pdo';//连接方式
$_xhprof['servername'] = 'myserver';//服务器名
$_xhprof['namespace'] = 'www97';/
$_xhprof['url'] = 'http://xhprof/xhprof_html';//想URL地址
/*
 * MySQL/MySQLi/PDO ONLY
 * Switch to JSON for better performance and support for larger profiler data sets.
 * WARNING: Will break with existing profile data, you will need to TRUNCATE the profile data table.
 */
$_xhprof['serializer'] = 'php';

//Uncomment one of these, platform dependent. You may need to tune for your specific environment, but they're worth a try

//These are good for Windows
/*
$_xhprof['dot_binary']  = 'C:\\Programme\\Graphviz\\bin\\dot.exe';
$_xhprof['dot_tempdir'] = 'C:\\WINDOWS\\Temp';
$_xhprof['dot_errfile'] = 'C:\\WINDOWS\\Temp\\xh_dot.err';
*/

//These are good for linux and its derivatives.

$_xhprof['dot_binary']  = '/usr/bin/dot';
$_xhprof['dot_tempdir'] = '/tmp';
$_xhprof['dot_errfile'] = '/tmp/xh_dot.err';


$ignoreURLs = array();

$ignoreDomains = array();

$exceptionURLs = array();

$exceptionPostURLs = array();
$exceptionPostURLs[] = "login";


$_xhprof['display'] = false;
$_xhprof['doprofile'] = false;

//Control IPs allow you to specify which IPs will be permitted to control when profiling is on or off within your application, and view the results via the UI.
//控制IP地址允许您指定哪些IP将被允许分析是你的应用程序中打开或关闭时控制，并通过用户界面查看结果。
$controlIPs = false; //Disables access controlls completely.
//$controlIPs = array();
//$controlIPs[] = "127.0.0.1";   // localhost, you'll want to add your own ip here 本地主机，你会想在这里添加你自己的IP
//$controlIPs[] = "::1";         // localhost IP v6 本地主机的IPv6

//$otherURLS = array();

// ignore builtin functions and call_user_func* during profiling 忽略分析期间内建函数和call_user_func*
//$ignoredFunctions = array('call_user_func', 'call_user_func_array', 'socket_select');

//Default weight - can be overidden by an Apache environment variable 'xhprof_weight' for domain-specific values
//默认重 - 可通过一个Apache环境变量特定于域的值所覆盖'XHProf的重量“
$weight = 100;

if($domain_weight = getenv('xhprof_weight')) {
        $weight = $domain_weight;
}

unset($domain_weight);

  /**
  * The goal of this function is to accept the URL for a resource, and return a "simplified" version
  * thereof. Similar URLs should become identical. Consider:
  * http://example.org/stories.php?id=2323
  * http://example.org/stories.php?id=2324
  * Under most setups these two URLs, while unique, will have an identical execution path, thus it's
  * worthwhile to consider them as identical. The script will store both the original URL and the
  * Simplified URL for display and comparison purposes. A good simplified URL would be:
  * http://example.org/stories.php?id=
  *
  * @param string $url The URL to be simplified
  * @return string The simplified URL
  */
  function _urlSimilartor($url)
  {
      //This is an example
      $url = preg_replace("!\d{4}!", "", $url);

      // For domain-specific configuration, you can use Apache setEnv xhprof_urlSimilartor_include [some_php_file]
      if($similartorinclude = getenv('xhprof_urlSimilartor_include')) {
        require_once($similartorinclude);
      }

      $url = preg_replace("![?&]_profile=\d!", "", $url);
      return $url;
  }

  function _aggregateCalls($calls, $rules = null)
  {
    $rules = array(
        'Loading' => 'load::',
        'mysql' => 'mysql_'
        );

    // For domain-specific configuration, you can use Apache setEnv xhprof_aggregateCalls_include [some_php_file]
        if(isset($run_details['aggregateCalls_include']) && strlen($run_details['aggregateCalls_include']) > 1)
                {
        require_once($run_details['aggregateCalls_include']);
                }

    $addIns = array();
    foreach($calls as $index => $call)
    {
        foreach($rules as $rule => $search)
        {
            if (strpos($call['fn'], $search) !== false)
            {
                if (isset($addIns[$search]))
                {
                    unset($call['fn']);
                    foreach($call as $k => $v)
                    {
                        $addIns[$search][$k] += $v;
                    }
                }else
                {
                    $call['fn'] = $rule;
                    $addIns[$search] = $call;
                }
                unset($calls[$index]);  //Remove it from the listing
                break;  //We don't need to run any more rules on this
            }else
            {
                //echo "nomatch for $search in {$call['fn']}<br />\n";
            }
        }
    }
    return array_merge($addIns, $calls);
  }
