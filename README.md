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

<p>Сниппеты должны придерживаться правил</p>

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
