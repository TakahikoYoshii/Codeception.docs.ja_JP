# Webサービスをテストする

The same way we tested a web site, Codeception allows you to test web services. They are very hard to test manually, so it's really good idea to automate web service testing. As a standards we have SOAP and REST, which are represented in corresponding modules. We will cover them in this chapter.

CodeceptionはWebサイトのテストと同じ方法で、Webサービスをテストすることができます。Webサービスを手動でテストすることはとても大変なので、テストを自動化することはとても良いアイディアです。CodeceptionにはSOAPとRESTに対応したモジュールが標準で備えています。この章ではそれらのモジュールについて説明します。

You should start with creating a new test suite, which was not provided by the `bootstrap` command. We recommend to call it **api** and use the `ApiTester` class for it.

新しくテストスイートを作成するところからはじめましょう。これは `bootstrap` コマンドでは提供されていません。テストスイートの名前は **api** とし、`ApiTester` クラスを使いましょう。

```bash
$ php codecept.phar generate:suite api
```

We will put all the api tests there.

ここにAPIのテストを記述していきます。

## REST

The REST web service is accessed via HTTP with standard methods: `GET`, `POST`, `PUT`, `DELETE`. They allow to receive and manipulate entities from the service. Accessing WebService requires HTTP client, so for using it you need the module `PhpBrowser` or one of framework modules set up. For example, we can use the `Symfony2` module for Symfony2 applications in order to ignore web server and test web service internally.

REST方式のWebサービスは、HTTPの標準的なメソッドである `GET`、`POST`、`PUT`、`DELETE` を介してアクセスされます。これにより、Webサービスからエンティティを受け取り、操作することができます。WebサービスへのアクセスにはHTTPクライアントが必要であるため、`PhpBrowser` やいずれかのフレームワーク用モジュールのセットアップを行う必要があります。Webサーバーを無視し、Webサービスを内部的にテストするために、たとえば、Symfony2 で実装されたアプリケーションであれば、`Symfony2` モジュールを使用します。

Configure modules in `api.suite.yml`:

`api.suite.yml` にモジュールの設定を行います。

``` yaml
class_name: ApiTester
modules:
    enabled: [PhpBrowser, REST, ApiHelper]
    config:
		PhpBrowser:
			url: http://serviceapp/
		REST:
		    url: http://serviceapp/api/v1/
```

The REST module will automatically connect to `PhpBrowser`. In case you provide it with Symfony2, Laravel4, Zend, or other framework module, it will connect to them as well. Don't forget to run the `build` command once you finished editing configuration.

RESTモジュールは自動的に `PhpBrowser` に接続します。Symfony2、Laravel4、Zendやそのほかのフレームワークモジュールによって提供する場合においても同様に接続します。設定の編集が完了したら、`build` コマンドを実行するのを忘れないでください。

Let's create the first sample test:

それでは最初のテストを作成しましょう。

```bash
$ php codecept.phar generate:cept api CreateUser
```

It will be called `CreateUserCept.php`. We can use it to test creation of user via web service.

これを `CreateUserCept.php` と呼ぶこととします。Webサービスを介してのユーザーの作成をテストするために使用します。

`CreateUserCept.php`

```php
<?php
$I = new ApiTester($scenario);
$I->wantTo('create a user via API');
$I->amHttpAuthenticated('service_user', '123456');
$I->haveHttpHeader('Content-Type', 'application/x-www-form-urlencoded');
$I->sendPOST('users', ['name' => 'davert', 'email' => 'davert@codeception.com']);
$I->seeResponseCodeIs(200);
$I->seeResponseIsJson();
$I->seeResponseContains('{"result":"ok"}');
?>
```

REST module is designed to be used with services that serve responses in JSON format. For example, method `seeResponseContainsJson` will convert provided array to JSON and check whether response contains it.

RESTモジュールはJSON形式をレスポンスするWebサービスを扱えるよう設計されています。たとえば、seeResponseContainsJson` メソッドは与えられた配列をJSON形式に変換し、それがレスポンスに含まれているかどうかをチェックします。

You may want to perform more complex assertions on response. This can be done with writing your own methods in [ヘルパー](http://codeception.com/docs/03-ModulesAndHelpers#Helpers) classes. To access the latest JSON response you will need to get `response` property of `REST` module. Let's demonstrate it with `seeResponseIsHtml` method:

レスポンスに対して、より複雑な検証を行いたい場合があると思います。そのためには [ヘルパー](http://codeception.com/docs/03-ModulesAndHelpers#Helpers) クラスに独自のメソッドを記述します。最後のJSONレスポンスにアクセスするためには、`REST` モジュールの `response` プロパティーを使用します。次に示す `seeResponseIsHtml` メソッドで説明しましょう。

```php
<?php
class ApiHelper extends \Codeception\Module
{
	public function seeResponseIsHtml()
	{
		$response = $this->getModule('REST')->response;
        \PHPUnit_Framework_Assert::assertRegex('~^<!DOCTYPE HTML(.*?)<html>.*?<\/html>~m', $response);
	}
}
?>
```

The same way you can receive request parameters and headers.

同じ方法で、リクエストパラメーターや、ヘッダー情報を取得することができます。

## SOAP

SOAP web services are usually more complex. You will need PHP [configured with SOAP support](http://php.net/manual/en/soap.installation.php). Good knowledge of XML is required too. `SOAP` module uses specially formatted POST request to connect to WSDL web services. Codeception uses `PhpBrowser` or one of framework modules to perform interactions. If you choose using a framework module, SOAP will automatically connect to the underliying framework. That may improve the speed of a test execution and will provide you with more detailed stack traces.

SOAP方式のWebサービスは通常、より複雑になります。[SOAPサポートを有効にする](http://php.net/manual/ja/soap.installation.php)必要があります。XML関する十分な知識も必要とされます。`SOAP` モジュールは、WSDLで表されたWebサービスに接続するために、特別な形式のPOSTリクエストを利用します。Codeceptionは `PhpBrowser`やいずれかのフレームワーク用モジュールを用いてやり取りを行います。もしフレームワーク用モジュールを選択した場合、SOAPモジュールは自動的に基盤となるフレームワークに接続します。これにより、テスト実行の速度を向上させることができ、詳細なスタックトレースを提供できるようになります。

Let's configure `SOAP` module to be used with `PhpBrowser`:

それでは `PhpBrowser` とともに使用する `SOAP` モジュールを設定しましょう。

``` yaml
class_name: ApiTester
modules:
    enabled: [PhpBrowser, SOAP, ApiHelper]
    config:
		PhpBrowser:
			url: http://serviceapp/
		SOAP:
		    endpoint: http://serviceapp/api/v1/
```

SOAP request may contain application specific information, like authentication or payment. This information is provided with SOAP header inside the `<soap:Header>` element of XML request. In case you need to submit such header, you can use `haveSoapHeader` action. For example, next line of code

SOAPリクエストには認証や支払いのようなアプリケーション固有の情報を含みます。この情報はXMLリクエストの `<soap:Header>` 要素に含まれるSOAPヘッダーによって提供されます。もしこのようなヘッダーを送信したい場合、`haveSoapHeader` メソッドを使用することができます。たとえば次のようになります。

```php
<?php
$I->haveSoapHeader('Auth', array('username' => 'Miles', 'password' => '123456'));
?>
```
will produce this XML header

このコードは次のXMLヘッダーを生成します。


```xml
<soap:Header>
<Auth>
	<username>Miles</username>
	<password>123456</password>
</Auth>
</soap:Header>
```

Use `sendSoapRequest` method to define the body of your request.

リクエストのボディを定義するためには `sendSoapRequest` を使用します。

```php
<?php
$I->sendSoapRequest('CreateUser', '<name>Miles Davis</name><email>miles@davis.com</email>');
?>
```

This call will be translated to XML:

この呼び出しは次のXMLに変換されます。

```xml
<soap:Body>
<ns:CreateUser>
	<name>Miles Davis</name>
	<email>miles@davis.com</email>
</ns:CreateUser>
</soap:Body>
```

And here is the list of sample assertions that can be used with SOAP.

そして、SOAPモジュールで使用できるアサーションの一覧がこちらです。


```php
<?php
$I->seeSoapResponseEquals('<?xml version="1.0"?><error>500</error>');
$I->seeSoapResponseIncludes('<result>1</result>');
$I->seeSoapResponseContainsStructure('<user><name></name><email></email>');
$I->seeSoapResponseContainsXPath('//result/user/name[@id=1]');
?>
```

In case you don't want to write long XML strings, consider using [XmlBuilder](http://codeception.com/docs/reference/XmlBuilder) class. It will help you to build complex XMLs in jQuery-like style.
In the next example we will use `XmlBuilder` (created from SoapUtils factory) instead of regular XMLs.

もし長いXMLを記述したくない場合、[XmlBuilder](http://codeception.com/docs/reference/XmlBuilder) クラスの利用を考えてみてください。これはjQueryのようなスタイルで複雑なXMLを構築するのに役に立ちます。
次の例では通常のXMLに替えて、（SoapUtilsファクトリによって作成された）`XmlBuilder` を使用しています。

```php
<?php
use \Codeception\Util\Soap;

$I = new ApiTester($scenario);
$I->wantTo('create user');
$I->haveSoapHeader('Session', array('token' => '123456'));
$I->sendSoapRequest('CreateUser', Soap::request()
	->user->email->val('miles@davis.com'));
$I->seeSoapResponseIncludes(Soap::response()
	->result->val('Ok')
		->user->attr('id', 1)
);
?>
```

It's up to you to decide whether to use `XmlBuilder` or plain XML. `XmlBuilder` will return XML string as well.
`XmlBuilder` を使うか、プレーンなXMLを利用するかは、どちらでも構いません。`XmlBuilder` も同様にXML文字列を返します。

You may extend current functionality by using `SOAP` module in your helper class. To access the SOAP response as `\DOMDocument` you can use `response` property of `SOAP` module.

ヘルパークラスの中で`SOAP`モジュールを利用することにより、機能を拡張することができます。`\DOMDocument` としてSOAPレスポンスにアクセスするためには、`SOAP` モジュールの `response` プロパティーを使用します。

```php
<?php
class ApiHelper extends \Codeception\Module {

	public function seeResponseIsValidOnSchema($schema)
	{
		$response = $this->getModule('SOAP')->response;
		$this->assertTrue($response->schemaValidate($schema));
	}
}
?>
```

## Conclusion

Codeception has two modules that will help you to test various web services. They need a new `api` suite to be created. Remember, you are not limited to test only response body. By including `Db` module you may check if a user has been created after the `CreateUser` call. You can improve testing scenarios by using REST or SOAP responses in your helper methods.

Codeceptionは様々なWebサービスをテストするために役立つモジュールを２つ備えています。それらを利用するために 新しく `api` スイートを作成する必要がありました。レスポンスボディだけのテストしかできないわけではないことを覚えておいてください。`Db` モジュールを使用することで、`CreateUser` の呼び出し後にユーザーが作成されているかどうかテストすることができます。ヘルパーメソッドを利用することでRESTやSOAPを使ったテストシナリオを向上することができます。