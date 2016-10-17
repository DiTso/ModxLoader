# ModxLoader

<h4>Плагин с классом Loader</h4>

<p>Плагин позволяет обращаться к сниппетам как через ajax так и напрямую</p>
<p>Для установки нужно создать плагин Loader с ниже указанным кодом и на события OnWebPageInit и OnPageNotFound</p>
<pre>
require MODX_BASE_PATH.'assets/plugins/loader/plugin.loader.php';
</pre>

<p>
пример вызова своего сниппета в php
</p>
<pre>
$modx->load->controller('account/controller/login', $config);
</pre>
<p>
<b>'account/controller/login'</b> - путь до нужного сниппета
</p>
<p>
<p><b>$config</b> - массив с передаваемыми параметрами
</p>

<br>
<p><b>AJAX</b></p>
```js
$.ajax({
    url: 'ajax?route=account/controller/login/ajax',
    dataType: 'json',
    type: 'post',
    data: params,
    success: function(json) {

	// ваш код

    },
    error: function(xhr, ajaxOptions, thrownError) {
	alert(thrownError + "\r\n" + xhr.statusText + "\r\n" + xhr.responseText);
    }
});
```

<br>
<p><b>PHP</b></p>
<p>Сниппеты должны придерживаться правил и находится в папке 'assets/snippets/'</p>
<p>ControllerAccountControllerLogin - Controller + 'account/controller/login'</p>

```php
<?php
class ControllerAccountControllerLogin extends Loader {

	public function index() {
	
		// ваш код
		
	}
	
}
?>
```
<br>
Рабочий код можно посмотреть в сниппете ModxAccount

PHP https://github.com/64j/ModxAccount/blob/master/account/controller/login.php

JS ajax - https://github.com/64j/ModxAccount/blob/master/account/view/login.tpl#L57

<br>
<br>
<br>
<p>Также можно установить плагин без инклюда, описанного выше. </p>
<h4 id="install_modxLoader">Установка:</h4>
<p>События плагина <b>OnWebPageInit</b> и <b>OnPageNotFound</b></p>
```php

// OnWebPageInit OnPageNotFound

class Loader {
	private $registry;
	private $data = array();

	public function __construct($modx) {
		$this->modx = $modx;
	}

	public function __get($key) {
		return (isset($this->data[$key]) ? $this->data[$key] : null);
	}

	public function __set($key, $value) {
		$this->data[$key] = $value;
	}

	public function controller($route, $data = array()) {
		$parts = preg_replace(array('/\.*[\/|\\\]/i','/[\/|\\\]+/i'), array('/', '/'), (string) $route);
		while($parts) {
			$file = MODX_BASE_PATH . 'assets/snippets/' . implode('/', $parts) . '.php';
			$class = 'Controller' . preg_replace('/[^a-zA-Z0-9]/', '', implode('/', $parts));
			if(is_file($file)) {
				include_once($file);
				break;
			} else {
				$method = array_pop($parts);
			}
		}
		if(!isset($method)) {
			$method = 'index';
		}
		if(substr($method, 0, 2) == '__') {
			return false;
		}
		if($method == 'index' && isset($_REQUEST['route'])) {
			$this->modx->sendRedirect($this->modx->config['site_url']);
		}
		$output = '';
		if(class_exists($class)) {
			$controller = new $class($this->modx);
			if(is_callable(array(
				$controller,
				$method
			))) {
				$output = call_user_func(array(
					$controller,
					$method
				), $data);
			} else {
				$this->modx->sendRedirect($this->modx->config['site_url']);
			}
		} else {
			$this->modx->sendRedirect($this->modx->config['site_url']);
		}
		return $output;
	}

	public function view($template, $data = array()) {
		$file = MODX_BASE_PATH . $template;
		if(file_exists($file)) {
			extract($data);
			ob_start();
			require($file);
			$output = ob_get_contents();
			ob_end_clean();
		} else {
			trigger_error('Error: Could not load template ' . $file . '!');
			exit();
		}
		return $output;
	}
}

$modx->load = new Loader($modx);

if($modx->Event->name == 'OnPageNotFound') {
	if($_REQUEST['q'] == 'ajax' && isset($_REQUEST['route'])) {
		$modx->load->controller($_REQUEST['route'], $_REQUEST, true);
		exit;
	}
}
```
<br>
<br>
<p>Вместо плагина на событие <b>OnPageNotFound</b> можно использовать <a href="http://modx.im/blog/triks/2103.html" target="_blank">Ajax метод 4</a> от <a href="https://github.com/AgelxNash" target="_blank">Agel_Nash</a></p>
<h4 id="ajax_method_4"><b>ajax.php</b></h4>
```php
<?php

define('MODX_API_MODE', true);

include_once(dirname(__FILE__) . "/index.php");

$modx->db->connect();

if (empty ($modx->config)) {
    $modx->getSettings();
}

$modx->invokeEvent("OnWebPageInit");

if(!isset($_SERVER['HTTP_X_REQUESTED_WITH']) || (strtolower($_SERVER['HTTP_X_REQUESTED_WITH']) != 'xmlhttprequest') || $_SERVER['REQUEST_METHOD'] != 'POST'){
	$modx->sendRedirect($modx->config['site_url']);
}

$json = array();

$json = $modx->load->controller($_REQUEST['route'], $_REQUEST);

header('content-type: application/json');
echo json_encode($json);
```
