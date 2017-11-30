## チェックボックスの印刷

4Dでは「フォームにフィールドや変数を配置する」という直感的なフィーリングでユーザーインタフェースや帳票を作成することができます。この場合，フォームオブジェクト（ボタン・テキスト入力エリア・リストボックスなど）にデータソース（フィールド・変数・式）を関連づけるという操作をまとめて実行していることになります。つまり，厳密にいえば，オブジェクトとデータは別々のものであり，連動している（例：ボタンをクリックしたらフィールドに値が代入される・フィールドに値を代入したらボタンがクリックされた状態で表示される）のは，プロパティ設定によって両者が関連づけられているからです。

チェックボックスは，本質的にはボタンオブジェクトの一種であり，画面上ではボタンのように振る舞います。通常のボタンと違うのは，クリックされた状態が持続する点です。いずれにしても，ボタンは押されていない状態がデフォルトなので，フォームの配置されたチェックボックスは，値が``0``または``False``の状態からスタートします。

たとえば，``CB``というデータソースがチェックボックスにバインドされている場合，下記のようなコードを実行すると，フォームが表示されると同時に``CB``には``False``が代入されることになります。

```
C_BOOLEAN(CB)
CB:=True

$w:=Open form window("boxes")
DIALOG("boxes")
CLOSE WINDOW($w)
```

仕様として明記されているわけではいませんが，検証してみると，この代入は，``On Load``イベントよりも前（``FORM LOAD``と同時）に実行されていることがわかります。さらに，カレントレコードがフォームに反映されるのは代入の後であることも推察できます。

つまり，チェックボックスが配置された詳細フォームを表示または印刷する場合，チェックボックスのデータソースがフィールドであれば，フォームの表示によってカレントレコードの値が書き換えられることはなく，フィールドが``1``または``True``であれば，チェックされたボックスが表示または印刷されますが，チェックボックスのデータソースが変数の場合，必ずチェックされていないボックスが表示または印刷されるということです。これは，チェックボックスがボタンの一種であることに由来する「当然の成り行き」といえるかもしれません。

* 図1. 変数チェックボックスの表示

<img width="98" alt="2017-11-30 12 38 54" src="https://user-images.githubusercontent.com/10509075/33411908-9bde774c-d5cb-11e7-8629-58e46dab8207.png">

* 図2. フィールドチェックボックスの表示

<img width="98" alt="2017-11-30 12 41 32" src="https://user-images.githubusercontent.com/10509075/33411929-d0df8828-d5cb-11e7-9612-ad6b8f9f77c9.png">

* 図3. 変数チェックボックスの印刷

<img width="135" alt="2017-11-30 12 38 43" src="https://user-images.githubusercontent.com/10509075/33411914-b0839c90-d5cb-11e7-9f75-93da4671a334.png">


* 図4. フィールドチェックボックスの印刷

<img width="135" alt="2017-11-30 12 42 43" src="https://user-images.githubusercontent.com/10509075/33411959-fcbcc5be-d5cb-11e7-9f16-9f7e21a9620d.png">

**注記**: 「図3. 変数チェックボックスの印刷」では，テキスト入力エリアの「式」に``String``関数が設定されています。この場合，印刷時にはまず最初に一度だけ式が評価されるようです。そのため，チェックボックスが干渉する前の値，つまり``True``が出力されています。「式」に変数名をただ設定した場合，チェックボックスが干渉した後の値，つまり``False``が出力されました（同じ変数が評価のタイミングにより，別々の値で出力されている点に注目）。

<img width="130" alt="2017-11-30 12 53 45" src="https://user-images.githubusercontent.com/10509075/33412258-9c50f16c-d5cd-11e7-903e-a85766382219.png">

**ポイント**: 変数の場合，チェックボックスの初期状態（チェックされてない）がフォームの表示や印刷と同時に代入されてしまう。フィールドであれば問題ない。

### 変数チェックボックスを表示または印刷するには

変数をチェックボックスとして表示または印刷するためには，チェックボックス（ボタンの一種）の初期状態がデータソースに代入されることを防止しなければなりません。具体的には，チェックボックスのデータソース（式）を空にすることで，オブジェクトと変数を「暗示的に」関連付けしないことが必要です。

もちろん，それだけではチェックボックスのデータソース設定が復元できません。ひとつのアイデアとして，現バージョンではオブジェクト名のサイズが``255``文字まで拡張されているので，ここにデータソース情報を含めることができます。

* 図5. オブジェクト名にデータソース情報を含めた例

<img width="515" alt="2017-11-30 13 34 19" src="https://user-images.githubusercontent.com/10509075/33413152-3199b5ce-d5d3-11e7-831d-77b3cf1b76af.png">

この例では，``:``の後に続くのが変数名，となっています。フォームが表示または印刷された時点ではデータソースが未定義であり，後からデータソースとして変数を設定しているので，変数が「チェックされてない状態」に初期化されてしまうことを防ぐことができます。

```
If (Form event=On Load) | (Form event=On Printing Detail)
	
	FORM GET OBJECTS($names;$objects)
	For ($i;1;Size of array($names))
		$name:=$names{$i}
		$object:=$objects{$i}
		
		ARRAY LONGINT($pos;0)
		ARRAY LONGINT($len;0)
		
		If (Match regex("[^:]+:(.+)";$name;1;$pos;$len))
			$variable:=Get pointer(Substring($name;$pos{1};$len{1}))
			If (Not(Nil($object)))
				If (Not(Nil($variable)))
					OBJECT SET DATA SOURCE(*;$name;$variable)
				End if 
			End if 
		End if 
		
	End for 
	
End if 
```

* 図6. 変数チェックボックスの表示（修正後）

<img width="176" alt="2017-11-30 13 35 01" src="https://user-images.githubusercontent.com/10509075/33413182-6589f718-d5d3-11e7-8262-babef789a602.png">

* 図7. 変数チェックボックスの印刷（修正後）

<img width="265" alt="2017-11-30 13 35 08" src="https://user-images.githubusercontent.com/10509075/33413186-6deb1720-d5d3-11e7-9fc4-3cb1b484c210.png">

