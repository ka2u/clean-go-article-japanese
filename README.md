許可をいただいて以下の記事を翻訳しました

[GitHub - Pungyeon/clean-go-article: A reference for the Go community that covers the fundamentals of writing clean code and discusses concrete refactoring examples specific to Go.](https://github.com/Pungyeon/clean-go-article)


# Clean Go Code

## Preface: Why Write Clean Code?

このドキュメントは、開発者がよりクリーンなコードを作成できるようにすることを目的としたGoコミュニティへのリファレンスです。 個人のプロジェクトで作業している場合でも、より大きなチームの一員として作業している場合でも、クリーンなコードを書くことは重要なスキルです。 クリーンなコードを記述するための優れたパラダイムと一貫性のあるアクセス可能な標準を確立することは、開発者が自分の（または他者の）作業を理解しようとして無意味な時間を浪費するのを防ぐのに役立ちます。

> We don’t read code, we decode it – Peter Seibel

開発者として、ベストプラクティスを考慮せずに、当面便利な方法でコードを記述したい場合があります。 これにより、コードのレビューとテストがより困難になります。 ある意味では、私たちはエンコードしています。そして、そうすることで、他の人が自分の作品をデコードするのをより難しくしています。 しかし、コードを使いやすく、読みやすく、メンテナンスしやすくしたいのです。 そして、簡単な方法ではなく、正しい方法でコーディングする必要があります。

このドキュメントは、クリーンなコードを書くための基礎についての簡単で短い紹介から始まります。 後で、Goに固有の具体的なリファクタリングの例について説明します。


**A short word on** `gofmt`


`gofmt` に対する私の立場を明確にするために、いくつかのセンテンスを取り上げたいと思います。なぜなら、このツールに関しては、私が反対していることがたくさんあるからです。 私はキャメルケースの場合よりもスネークケースの場合を好み、定数が大文字であることが非常に好きです。 そして、当然、ブラケットの配置についても多くの意見があります。 そうは言っても、 `gofmt` を使用すると、Goコードを記述するための共通の標準フォーマットを使用できます。これは素晴らしいことです。 開発者でもある自分自身、Goプログラマーは `gofmt` によって多少制限されていると感じていると思います。特に、その標準フォーマットのルールのいくつかに同意しない場合は。 しかし、私の意見では、同種のコードは完全な表現の自由を持つよりも重要です。

## Table of Contents

* [Introduction to Clean Code](#Introduction-to-Clean-Code)
    * [Test-Driven Development](#Test-Driven-Development)
    * [Naming Conventions](#Naming-Conventions)
      * [Comments](#Comments)
    	* [Function Naming](#Function-Naming)
    	* [Variable Naming](#Variable-Naming)
    * [Cleaning Functions](#Cleaning-Functions)
      * [Function Length](#Function-Length)
      * [Function Signatures](#Function-Signatures)
    * [Variable Scope](#Variable-Scope)
    * [Variable Declaration](#Variable-Declaration)
    
* [Clean Go](#Clean-Go)
    * [Return Values](#Return-Values) 
      * [Returning Defined Errors](#Returning-Defined-Errors)
      * [Returning Dynamic Errors](#Returning-Dynamic-Errors)
    * [Pointers in Go](#Pointers-in-Go)
    * [Closures Are Function Pointers](#Closures-are-Function-Pointers)
    * [Interfaces in Go](#Interfaces-in-Go)
    * [The Empty `interface{}`](#The-Empty-Interface)
* [Summary](#Summary)


## Introduction to Clean Code

クリーンコードは、読みやすくて保守可能なソフトウェアを促進する実用的な概念です。 クリーンなコードは、コードベースに対する信頼を確立し、不注意なバグが導入される可能性を最小限に抑えます。 また、開発者の開発速度を維持するのにも役立ちます。通常、バグを入れるリスクが高まるため、コードベースが拡大するにつれて急落します。


### Test-Driven Development

テスト駆動開発とは、短い開発サイクルまたはスプリントを通じてコードを頻繁にテストすることです。 最終的には、開発者にコードの機能と目的を問うことで、コードのクリーン化に貢献します。 テストを簡単にするために、開発者は1つのことだけを行う短い関数を作成することをお勧めします。 たとえば、40行よりも4行だけの関数をテスト（および理解）する方が間違いなく簡単です。


テスト駆動開発は、次のサイクルで構成されます。

1. テストを書く(またはテストを実行する)
2. テストがコケたら、通るようにテストを直す
3. リファクタリングする
4. 繰り返す

このプロセスにはテストとリファクタリングが絡み合っています。 コードをリファクタリングしてわかりやすく、または保守しやすくするために、変更を徹底的にテストして、関数の動作が変更されていないことを確認する必要があります。 これは、コードベースが大きくなるにつれて非常に便利です。

### Naming Conventions

#### Comments

最初にコードのコメントについて取り組みたいと思います。これは必須のプラクティスですが、誤用される傾向があります。 不要なコメントは、不適切な命名規則の使用など、基礎となるコードの問題を示している可能性があります。 ただし、特定のコメントが「必要」であるかどうかは多少主観的であり、コードがどの程度読みやすいかに依存します。 たとえば、適切に記述されたコードのロジックだが非常に複雑なため、何が起こっているのかを明確にするためにコメントが必要になる場合があります。 その場合、コメントは有用であり、したがって必要であると主張するかもしれません。


Goでは、`gofmt` により、すべてのパブリック変数と関数に注釈を付ける必要があります。これはコードを文書化するための一貫したルールが得られるので、私はとても良いことだと思います。 ただし、自動生成されたコメントと他のすべてのコメントを常に区別したいと思います。 ドキュメント用の注釈コメントは、ドキュメントのように記述する必要があります。高レベルの抽象化を行い、コードの論理的な実装内容をできる限り少なくする必要があります。


何故ならば、コードが包括的かつ表現力豊かに書かれていると、それ自体がコードを説明しているからです。コードがこれらのいずれでもない場合、複雑なロジックを説明するコメントを導入することが受け入れられると考える人もいます。 残念ながら、それは実際には役に立ちません。 1つは、コードをレビューするのを非常に邪魔する傾向があるため、ほとんどの人はコメントを読みません。さらに、ご想像のとおり、開発者は、コメントであふれている不明瞭なコードのレビューを余儀なくされても、あまり満足しません。 あなたのコードが何をしているのかを理解するために人々が読む必要のあるものが少なければ少ないほど良いでしょう。

一歩退いて、いくつかの具体的な例を見てみましょう。 してはいけないコメントは次のとおりです:

```go
// 0から9の間でイテレートして
// 毎回 doSomething 関数を呼ぶ
for i := 0; i < 10; i++ {
	doSomething(i)
}
```

これは、私がチュートリアルコメントと呼んでいるものです。 チュートリアルではかなり一般的であり、多くの場合、言語の低レベルの機能（または一般的なプログラミング）について説明しています。 これらのコメントは初心者には役立つかもしれませんが、プロダクションコードではまったく役に立ちません。 願わくば、開発チームで作業を開始するまでにループ構造のような単純なものを理解していないプログラマーとは協力しないことを願っています。 プログラマーとして、何が起こっているかを理解するためにコメントを読む必要はありません。コードを単純に読むことができるので、0から9の範囲で繰り返していることを知っています。 したがって、以下のことわざ：

> Document why, not how. – Venkat Subramaniam

このロジックに従って、コメントを変更して、なぜ0〜9の範囲で反復するのかを説明します:

```go
// ワークロードをさばくために10個のスレッドを生成する
for i := 0; i < 10; i++ {
	doSomething(i)
}
```

これでループが発生する理由がわかり、コードを読むだけで何をしているのかがわかります...ちょっとだけ。

これは、私がクリーンコードと考えるものではありません。 コメントがよくないのは、コードがよく書かれていると仮定すると（このコードはそうではありません）おそらくそのような説明を文で表現する必要はないはずだからです。 技術的には、まだ私たちがやろうとしている理由ではなく、私たちがやっていることをまだ書いています。 より意味のある名前を使用すると、コードでこの「何」を直接簡単に表現できます。

```go
for workerID := 0; workerID < 10; workerID++ {
	instantiateThread(workerID)
}
```

変数名と関数名を少し変更するだけで、コードで直接実行していることを説明できました。 読む人はコメントを読み、文をコードにマップする必要がないので、読む人にとってははるかに明確です。 代わりに、彼らは単にコードを読んで、それが何をしているのかを理解することができます。

もちろん、これは比較的些細な例です。 残念ながら、明確で表現力豊かなコードを書くことは、必ずしも簡単なことではありません。 コードベース自体の複雑さが増すにつれて、ますます困難になる可能性があります。 この考え方でコメントを書く練習を重ね、何をしているのかを説明することを避ければ、コードはよりきれいになります。

##### Function Naming

関数の命名規則に移りましょう。 ここでの一般的なルールは本当に単純です。関数がより具体的であるほど、その名前はより一般的です。 つまり、一般的な機能を説明する `Run` や `Parse` などの非常に広く短い機能名から始めたいと考えています。 コンフィギュレーションパーサーを作成していると想像しましょう。 この命名規則に従って、抽象化の最上位レベルは次のようになります。

```go
func main() {
	configpath := flag.String("config-path", "", "configuration file path")
	flag.Parse()

	config, err := configuration.Parse(*configpath)

	...
}
```

`Parse` 関数の命名に焦点を当てます。 この関数は非常に短く一般的な名前ですが、実際に何を達成しようとしているのかは明確です。

関数の中を見ると、関数の命名がより具体的になります:

```go
func Parse(filepath string) (Config, error) {
	switch fileExtension(filepath) {
	case "json":
		return parseJSON(filepath)
	case "yaml":
		return parseYAML(filepath)
	case "toml":
		return parseTOML(filepath)
	default:
		return Config{}, ErrUnknownFileExtension
	}
}
```

ここでは、ネストされた関数呼び出しを過度に特定することなく、親から明確に区別しました。 これにより、ネストされた各関数呼び出しは、親のコンテキスト内だけでなく、それ自体でも意味をなします。 一方、 `parseJSON` 関数にjsonという名前を付けた場合、それだけでは機能しません。 機能は名前から失われ、この関数がJSONを解析、作成、またはマーシャリングするかどうかを判断できなくなります。

`fileExtension` は実際にはもう少し具体的であることに注意してください。 これは、その機能が非常に具体的であるためです。

```go
func fileExtension(filepath string) string {
	segments := strings.Split(filepath, ".")
	return segments[len(segments)-1]
}
```

関数名のこの種の論理的な進行（高度な抽象化からより具体的な抽象化まで）により、コードを追いやすくなります。 
代替案を検討してください：抽象化の最高レベルがあまりにも具体的である場合、`DetermineFileExtensionAndParseConfigurationFile` のようなすべてのベースをカバーしようとする名前になります。 これは読みにくいです。 明確にしようとしているにもかかわらず、あまりにも具体的になりすぎて、読む人を混乱させようとしています。

##### Variable Naming

興味深いことに、変数の場合は逆です。 関数とは異なり、ネストされたスコープに入るほど、変数の名前はより具体的なものからより具体的でないものに変更します。

> You shouldn’t name your variables after their types for the same reason you wouldn’t name your pets 'dog' or 'cat'. – Dave Cheney


関数のスコープ内を深く進むにつれて、変数名の具体性が低くなるのはなぜですか？ 簡単に言えば、変数のスコープが小さくなるにつれて、その変数が何を表しているのかが読む人にとってますます明確になり、それによって特定の命名の必要がなくなります。 前の関数  `fileExtension` の例では、必要に応じて変数 `segments` の名前を `s` に短縮することさえできます。 変数のコンテキストは非常に明確であるため、より長い変数名でこれ以上説明する必要はありません。 これの別の良い例は、ネストされたforループです:

```go
func PrintBrandsInList(brands []BeerBrand) {
	for _, b := range brands { 
		fmt.Println(b)
	}
}
```

上記の例では、変数 `b` のスコープは非常に小さいため、変数が正確に何を表しているかを覚えるのに頭を使う必要はありません。 ただし、`brands` の範囲はわずかに広いため、より具体的にするのに役立ちます。 以下の関数で変数スコープを展開すると、この区別がさらに明確になります:

```go
func BeerBrandListToBeerList(beerBrands []BeerBrand) []Beer {
	var beerList []Beer
	for _, brand := range beerBrands {
		for _, beer := range brand {
			beerList = append(beerList, beer)
		}
	}
	return beerList
}
```

すばらしい！ この関数は読みやすいです。 ここで、変数に名前を付けるときに反対の（つまり、間違った）ロジックを適用しましょう:

```go
func BeerBrandListToBeerList(b []BeerBrand) []Beer {
	var bl []Beer
	for _, beerBrand := range b {
		for _, beerBrandBeerName := range beerBrand {
			bl = append(bl, beerBrandBeerName)
		}
	}
	return bl
}
```

この関数が何をしているのかを理解することは可能ですが、変数名が過度に簡潔であるため、深く掘り下げるにつれて論理に従うのが難しくなります。 これは、短い変数名と長い変数名を混在させているため、混乱する可能性があります。

### Cleaning Functions

変数と関数に名前を付けるためのいくつかのベストプラクティスを知ったので、コメントを使用してコードを明確にし、関数をリファクタリングしてクリーンにするための方法の詳細に進みましょう。

#### Function Length

> How small should a function be? Smaller than that! – Robert C. Martin

クリーンコードを書くときの主な目標は、コードを簡単に消化できるようにすることです。 これを行う最も効果的な方法は、機能をできるだけ短くすることです。 コードの重複を避けるために必ずしもこれを行う必要はないことを理解することが重要です。 より重要な理由は、コードの理解度を向上させることです。


これをよりよく理解するには、関数の説明を非常に高いレベルで見ると役立ちます:

```
fn GetItem:
    - parse json input for order id
    - get user from context
    - check user has appropriate role
    - get order from database
```

短い関数（Goでは通常5〜8行）を記述することで、上記の説明とほぼ同じように自然に読み取るコードを作成できます。

```go
var (
	NullItem = Item{}
	ErrInsufficientPrivileges = errors.New("user does not have sufficient privileges")
)

func GetItem(ctx context.Context, json []bytes) (Item, error) {
	order, err := NewItemFromJSON(json)
	if err != nil {
		return NullItem, err
	}
	if !GetUserFromContext(ctx).IsAdmin() {
		return NullItem, ErrInsufficientPrivileges
	}
	return db.GetItem(order.ItemID)
}
```

小さな関数を使用すると、コードを記述する別の恐ろしい習慣であるインデント地獄もなくなります。 インデント地獄は通常、`if` ステートメントのチェーンが関数内で不注意にネストされている場合に発生します。 これにより、人間がコードを解析することは非常に難しくなり、発見されるたびに削除する必要があります。 インデント地獄は、 `interface{}` を使用して型キャストを使用する場合に特に一般的です:

```go
func GetItem(extension string) (Item, error) {
	if refIface, ok := db.ReferenceCache.Get(extension); ok {
		if ref, ok := refIface.(string); ok {
			if itemIface, ok := db.ItemCache.Get(ref); ok {
				if item, ok := itemIface.(Item); ok {
    			if item.Active {
						return Item, nil
					} else {
						return EmptyItem, errors.New("no active item found in cache")
 					}
				} else {
					eturn EmptyItem, errors.New("could not cast cache interface to Item")
				}
			} else {
				return EmptyItem, errors.New("extension was not found in cache reference")
 			}
		} else {
			return EmptyItem, errors.New("could not cast cache reference interface to Item")
		}
	}
	return EmptyItem, errors.New("reference not found in cache")
}
```

最初に、インデント地獄は、他の開発者がコードのフローを理解することを難しくします。 第二に、`if` ステートメントのロジックが拡張すると、どのステートメントが何を返すのかを判断するのが指数関数的に難しくなります（すべてのパスが何らかの値を返すことを保証する）。 さらに別の問題は、条件文のこの深いネストにより、読む人が頻繁にスクロールして、頭の多くの論理状態を追跡することを余儀なくされることです。 また、考慮しなければならない非常に多くの異なるネストされた可能性があるため、コードのテストとバグの検出がより困難になります。

開発者が上記のサンプルのような扱いにくいコードを絶えず解析しなければならない場合、インデント地獄は読む人を疲れさせることがあります。 当然のことながら、これは何としても避けたいものです。

それでは、この関数をどのようにきれいにするのでしょうか？ 幸いなことに、実際には非常に簡単です。 最初の反復では、できるだけ早くエラーを返すようにします。  `if` ステートメントと `else` ステートメントをネストする代わりに、いわば「コードを左にプッシュ」したいのです。 見てみましょう:

```go
func GetItem(extension string) (Item, error) {
	refIface, ok := db.ReferenceCache.Get(extension)
	if !ok {
		return EmptyItem, errors.New("reference not found in cache")
	}

	ref, ok := refIface.(string)
	if !ok {
    	// return cast error on reference 
	}

	itemIface, ok := db.ItemCache.Get(ref)
	if !ok {
		// return no item found in cache by reference
	}

	item, ok := itemIface.(Item)
	if !ok {
		// return cast error on item interface
	}

	if !item.Active {
		// return no item active
	}

	return Item, nil
}
```

関数をリファクタリングする最初の試みが完了したら、関数をより小さな関数に分割することができます。 良い経験則は次のとおりです。関数内で `value, err：=` パターンが複数回繰り返される場合、これはコードのロジックを小さな断片に分割できることを示しています。

```go
func GetItem(extension string) (Item, error) {
	ref, ok := getReference(extension)
	if !ok {
		return EmptyItem, ErrReferenceNotFound
	}
	return getItemByReference(ref)
}

func getReference(extension string) (string, bool) {
	refIface, ok := db.ReferenceCache.Get(extension)
	if !ok {
		return EmptyItem, false
	}
	return refIface.(string)
}

func getItemByReference(reference string) (Item, error) {
	item, ok := getItemFromCache(reference)
	if !item.Active || !ok {
    	return EmptyItem, ErrItemNotFound
	}
	return Item, nil
}

func getItemFromCache(reference string) (Item, bool) {
	if itemIface, ok := db.ItemCache.Get(ref); ok {
		return EmptyItem, false
	}
	return itemIface.(Item), true
}
```

前に述べたように、インデント地獄はコードのテストを困難にする可能性があります。  `GetItem` 関数をいくつかのヘルパーに分割すると、コードのテスト時にバグを追跡しやすくなります。 同じスコープ内の複数の `if` ステートメントで構成された元のバージョンとは異なり、 `GetItem` のリファクタリングされたバージョンには、考慮する必要がある分岐パスが2つだけになります。 ヘルパー関数も短くて読みやすくなっています。

> 注：プロダクションコードの場合、 `bool` 値の代わりにエラーを返すことにより、コードをさらに詳しく説明する必要があります。 これにより、エラーの発生元をより簡単に理解できます。 ただし、これらは単なる例の関数であるため、ここでは `bool` 値を返すだけで十分です。 エラーをより明示的に返す例については、後で詳しく説明します。


`GetItem` 関数をきれいにすると、全体的にコードの行が増えたことに注意してください。 ただし、コード自体は読みやすくなりました。 オニオンレイヤースタイルで階層化されており、興味のない「レイヤー」を無視して、調べたいものを単純に剥がすことができます。 これにより、一度に3〜5行しか読めないため、低レベルの機能を理解しやすくなります。

この例は、使用する行数ではコードのクリーン度を測定できないことを示しています。 コードの最初のバージョンは確かにずっと短かったです。 しかし、人為的に短く、読むのが非常に困難でした。 ほとんどの場合、コードをクリーニングすると、既存のコードベースから行数が増えます。 しかし、これは、複雑なロジックを持つという選択肢よりも非常に望ましい方法です。 これについて疑問がある場合は、次の関数についてどのように感じるかを考えてください。これは、コードはまったく同じですが、2行しか使用しません:

```go
func GetItemIfActive(extension string) (Item, error) {
	if refIface,ok := db.ReferenceCache.Get(extension); ok {if ref,ok := refIface.(string); ok { if itemIface,ok := db.ItemCache.Get(ref); ok { if item,ok := itemIface.(Item); ok { if item.Active { return Item,nil }}}}} return EmptyItem, errors.New("reference not found in cache")
}
```

#### Function Signatures

適切な関数名構造を作成すると、コードの意図を読みやすく理解しやすくなります。 上で見たように、関数を短くすると、関数のロジックを理解するのに役立ちます。 関数のクリーニングの最後の部分には、関数入力のコンテキストを理解することが含まれます。 これには、わかりやすい別のルールがあります。関数シグネチャには、1つまたは2つの入力パラメーターのみを含める必要があります。 特定の例外的なケースでは、3つでも問題ありませんが、ここでリファクタリングの検討を開始する必要があります。 関数の長さは5〜8行だけであるという規則と同様に、これは最初は非常に極端に思えます。 しかし、このルールは正当化する方がはるかに簡単だと感じています。

RabbitMQのGoライブラリのチュートリアルから次の関数を見てみます:

```go
q, err := ch.QueueDeclare(
	"hello", // name
	false,   // durable
	false,   // delete when unused
	false,   // exclusive
	false,   // no-wait
	nil,     // arguments
)
```

関数 `QueueDeclare` は6つの入力パラメーターを取りますが、これは非常に多いです。 いくらかの努力をすれば、コメントのおかげでこのコードが何をするのかを理解することができます。 ただし、コメントは実際には問題の一部です。前述のように、コメントは可能な限り説明的なコードに置き換える必要があります。 結局のところ、コメントなしで `QueueDeclare` 関数を呼び出すことを妨げるものは何もありません。

```go
q, err := ch.QueueDeclare("hello", false, false, false, false, nil)
```

ここで、コメント付きバージョンを見ずに、4番目と5番目の `false` の引数が何を表しているのかを思い出してください。 不可能ですよね？ ある時点で必然的に忘れるでしょう。 これにより、コストのかかるミスや修正が困難なバグが発生する可能性があります。 間違いは、誤ったコメント（間違った入力パラメーターのラベル付けを想像してください）によっても発生する場合があります。 この間違いを修正するのは、特にコードの熟知度が徐々に低下したり、最初から不十分だったりした場合、修正するのが耐え難いほど難しくなります。 したがって、これらの入力パラメーターを `Options` 構造体に置き換えることをお勧めします。

```go
type QueueOptions struct {
	Name string
	Durable bool
	DeleteOnExit bool
	Exclusive bool
	NoWait bool
	Arguments []interface{} 
}

q, err := ch.QueueDeclare(QueueOptions{
	Name: "hello",
	Durable: false,
	DeleteOnExit: false,
	Exclusive: false,
	NoWait: false,
	Arguments: nil,
})
```

これにより、コメントの誤用と変数に誤ってラベルを付けるという2つの問題が解決されます。 もちろん、プロパティを間違った値と混同することもできますが、これらの場合、コード内のどこに間違いがあるのかを判断するのははるかに簡単です。 プロパティの順序も問題にならないため、入力値の順序が間違っていても問題ありません。 この手法の最後に追加されたボーナスは、`QueueOptions` 構造体を使用して、関数の入力パラメーターの既定値を推測できることです。 Goの `struct` が宣言されると、すべてのプロパティがデフォルト値に初期化されます。 これは、 `QueueDeclare` オプションを実際に次の方法で呼び出すことができることを意味します:

```go
q, err := ch.QueueDeclare(QueueOptions{
	Name: "hello",
})
```

残りの値はデフォルト値の `false` に初期化されます（引数としては例外で、インターフェースのデフォルト値は `nil` です）。 このアプローチでは安全性が大幅に向上するだけでなく、意図が明確になります。 この場合、実際にはより少ないコードを書くことができます。 これは、プロジェクトの全員にとって万能な勝利です。


これに関する最後の注意：関数のシグネチャを変更することは常に可能とは限りません。 この場合、たとえば、実際には `QueueDeclare` 関数シグネチャはRabbitMQライブラリからのものであるため、実際には制御できません。 それは私たちのコードではないので、変更することはできません。 ただし、目的に合わせてこれらの関数をラップできます:

```go
type RMQChannel struct {
	channel *amqp.Channel
}

func (rmqch *RMQChannel) QueueDeclare(opts QueueOptions) (Queue, error) {
	return rmqch.channel.QueueDeclare(
		opts.Name,
		opts.Durable,
		opts.DeleteOnExit,
		opts.Exclusive,
		opts.NoWait,
		opts.Arguments, 
	)
} 
```

基本的に、`amqp.Channel` 型を含む `RMQChannel` という名前の新しい構造を作成します。これには、 `QueueDeclare` メソッドがあります。 次に、このメソッドの独自のバージョンを作成します。これは、基本的に、RabbitMQライブラリ関数の古いバージョンを呼び出すだけです。 新しい方法には、前述のすべての利点があり、RabbitMQライブラリのコードを実際に変更することなく、これを達成しました。

関数をラップするというこの考え方を使用して、後で `interface{}` について説明するときに、よりクリーンで安全なコードを紹介します。


### Variable Scope

それでは、一歩下がって、より小さな関数を書くという考えに立ち返りましょう。 これには、前の章では説明しなかったもう1つの素晴らしい副作用があります。より小さな関数を記述すると、通常、グローバルスコープにリークする可変変数への依存を排除できます。

グローバル変数には問題があり、クリーンなコードに属していません。 プログラマーが変数の現在の状態を理解することを非常に困難にします。 変数がグローバルで変更可能な場合、定義により、その値はコードベースの任意の部分で変更できます。 この変数が特定の値になることを保証することはできません...そしてそれは誰にとっても頭痛の種です。 これは、コードベースが拡大したときに悪化する些細な問題の別の例です。

大きなスコープを持つ非グローバル変数がどのように問題を引き起こす可能性があるかの短い例を見てみましょう。 これらの変数は、[Golang Scope Issue](https://idiallo.com/blog/golang-scopes) というタイトルの記事から取られたコードで示されているように、 *variable shadowning* の問題ももたらします:

```go
func doComplex() (string, error) {
	return "Success", nil
}

func main() {
	var val string
	num := 32

	switch num {
	case 16:
		// do nothing
	case 32:
		val, err := doComplex()
		if err != nil {
			panic(err)
		}
		if val == "" {
			// do something else
		}
	case 64:
		// do nothing
	}
    
	fmt.Println(val)
}
```

このコードの問題は何でしょう？ ざっと読むと、メイン関数の終わりまでに `var val` 文字列値がSuccessとして出力されるはずです。 残念ながら、これは事実ではありません。 この理由は次の行にあります:

```go
val, err := doComplex()
```

これは、`switch` の `case 32` スコープで新しい変数 `val` を宣言し、 `main` の最初の行で宣言された変数とは関係ありません。 もちろん、Goの構文はややトリッキーであると主張することができます。これは必ずしも同意するものではありませんが、手元にはさらに悪い問題があります。 可変で主にスコープが設定された変数としての `var val` 文字列の宣言は完全に不要です。 非常に簡単なリファクタリングを行うと、この問題は発生しなくなります:

```go
func getStringResult(num int) (string, error) {
	switch num {
	case 16:
		// do nothing
	case 32:
		return doComplex()
	case 64:
		// do nothing
	}
	return "" 
}

func main() {
	val, err := getStringResult(32)
	if err != nil {
		panic(err)
	}
	if val == "" {
		// do something else
	}
	fmt.Println(val)
}
```


リファクタリング後、`val` は変更されなくなり、スコープが縮小されました。 繰り返しになりますが、これらの関数は非常に単純であることに注意してください。 この種のコードスタイルがより大きく複雑なシステムの一部になると、エラーが発生した理由を把握することが不可能になる可能性があります。 私たちはこれが起こることを望んでいません。ソフトウェアエラーが一般的に嫌いであるだけでなく、同僚や私たち自身に失礼だからです。 このタイプのコードをデバッグするためにお互いの時間を浪費している可能性があります。 開発者は、Goのような特定の言語の変数宣言構文にこれらの問題を責めるのではなく、独自のコードに責任を負う必要があります。

余談ですが、 `// do something else` パートは `val` 変数を変更する別の試みです。そのロジックを、その前の部分だけでなく、独自の自己完結型関数として抽出する必要があります。 この方法では、変数の可変スコープを拡張する代わりに、新しい値を返すことができます:

```go
func getVal(num int) (string, error) {
	val, err := getStringResult(32)
	if err != nil {
		return "", err
	}
	if val == "" {
		return NewValue() // pretend function
	}
}

func main() {
	val, err := getVal(32)
	if err != nil {
		panic(err)
	}
	fmt.Println(val)
}
```

### Variable Declaration

変数のスコープと可変性に関する問題を回避する以外に、変数をできるだけ使用するところの近くに宣言することにより、読みやすさを改善することもできます。 Cプログラミングでは、変数を宣言するための次のアプローチがよく見られます:

```c
func main() {
	var err error
	var items []Item
	var sender, receiver chan Item
	
	items = store.GetItems()
	sender = make(chan Item)
	receiver = make(chan Item)
	
	for _, item := range items {
		...
	}
}
```

これには、変数スコープの説明で説明したのと同じ症状があります。 これらの変数は実際にはどの時点でも再割り当てされないかもしれませんが、この種のコーディングスタイルは、読む人は常に気をつけていなければならないので間違った方法です。 コンピューターの記憶と同じように、脳の短期記憶の容量は限られています。 どの変数が可変であり、コードの特定のフラグメントが変数を変更するかどうかを追跡しなければならないため、コードが何をしているのかを理解するのが難しくなります。 最終的に返される値を把握するのは悪夢です。 したがって、読む人（および将来の私たち）にとってこれを簡単にするために、変数をできるだけその使用するところに近いところで宣言することをお勧めします:

```go
func main() {
	var sender chan Item
	sender = make(chan Item)

	go func() {
		for {
			select {
			case item := <-sender:
				// do something
			}
		}
	}()
}
```

ただし、宣言の直後に関数を呼び出すことでさらに改善できます。 これにより、関数ロジックが宣言された変数に関連付けられていることがより明確になります:

```go
func main() {
	sender := func() chan Item {
		channel := make(chan Item)
		go func() {
			for {
				select { ... }
			}
		}()
		return channel
	}
}
```

そして、さらに変えると、匿名関数を移動して名前付き関数にすることができます:

```go
func main() {
	sender := NewSenderChannel()
}
  
func NewSenderChannel() chan Item {
	channel := make(chan Item)
	go func() {
		for {
			select { ... }
		}
	}()
	return channel
}
```

変数を宣言していること、および最初の例とは異なり、返されたチャネルに関連付けられているロジックが単純であることは依然として明らかです。 これにより、コードを走査して各変数の役割を理解しやすくなります。

もちろん、これは実際に `sender` 変数の変更を妨げるものではありません。 Goには `const struct` または `static` 変数を宣言する方法がないため、これについてできることは何もありません。 つまり、コードの後半でこの変数を変更しないようにする必要があります。

> 注：キーワードconstは存在しますが、使用がプリミティブ型のみに制限されています。


これを回避する方法の1つは、少なくとも変数の可変性をパッケージレベルに制限することです。 トリックには、変数をプライベートプロパティとして使用して構造体を作成することが含まれます。 このプライベートプロパティは、以降、このラッピング構造によって提供される他のメソッドを介してのみアクセス可能です。 チャンネルの例を拡張すると、これは次のようになります:

```go
type Sender struct {
	sender chan Item
}
  
func NewSender() *Sender {
	return &Sender{
		sender: NewSenderChannel(),
	}
}
  
func (s *Sender) Send(item Item) {
	s.sender <- item
}
```

これで、少なくともパッケージの外部からではなく、`Sender` 構造体の `sender` プロパティが変更されないことが保証されました。 このドキュメントを書いている時点では、これが公的に不変の非プリミティブ変数を作成する唯一の方法です。 少し冗長ですが、偶発的な変数の変更に起因する奇妙なバグに終わらないようにするための努力は本当に価値があります。

```go
func main() {
	sender := NewSender()
	sender.Send(&Item{})
}
```

上記の例を見ると、これがどのようにパッケージの使用を簡素化するかが明確です。 実装を隠すこの方法は、パッケージのメンテナーだけでなくユーザーにとっても有益です。 現在、 `Sender` 構造を初期化して使用する場合、その実装についての心配はありません。 これにより、アーキテクチャが大幅に緩和されます。 ユーザーは実装に関心がないので、ユーザーはパッケージとの連絡先を減らしているため、いつでも実装を変更できます。 パッケージでチャネルの実装を使用したくない場合は、Sendメソッドの使用を中断することなく、これを簡単に変更できます（現在の関数シグネチャに準拠している限り）。

注：AWS re：Invent 2017：世界を壊さずに変化を受け入れる（DEV319）のトークから、クライアントライブラリの抽象化を処理する方法について素晴らしい説明があります。

## Clean Go

このセクションでは、基礎となるクリーンコードの原則に重点を置いて、クリーンなGoコードを記述する一般的な側面よりも詳細に焦点を当てます。

### Return Values

#### Returning Defined Errors

エラーを返すクリーンな方法を説明することで、物事を簡単に始めることができます。 前に説明したように、クリーンなコードを書くことの主な目標は、コードベースの可読性、テスト容易性、保守性を確保することです。 ここで説明するエラーを返すための手法は、これら3つの目標すべてを非常に少ない労力で達成します。

カスタムエラーを返す通常の方法を考えてみましょう。 これは、 `Store` という名前のスレッドセーフなマップの実装から取った架空の例です:

```go
package smelly

func (store *Store) GetItem(id string) (Item, error) {
	store.mtx.Lock()
	defer store.mtx.Unlock()

	item, ok := store.items[id]
	if !ok {
		return Item{}, errors.New("item could not be found in the store") 
	}
	return item, nil
}
```


この関数を単独で検討する場合、この関数について本質的に悪い臭いはありません。  `Store` 構造体のアイテムマップを調べて、指定されたIDのアイテムが既にあるかどうかを確認します。 もしあるなら、それを返します。 そうでなければ、エラーを返します。 かなり標準的です。 それで、カスタムエラーを文字列値として返す問題は何でしょう？ さて、この関数を別のパッケージ内で使用するとどうなるか見てみましょう。

```go
func GetItemHandler(w http.ReponseWriter, r http.Request) {
	item, err := smelly.GetItem("123")
	if err != nil {
		if err.Error() == "item could not be found in the store" {
			http.Error(w, err.Error(), http.StatusNotFound)
			return
		}
		http.Error(w, errr.Error(), http.StatusInternalServerError)
		return
	} 
	json.NewEncoder(w).Encode(item)
}
```

これは実際にはそれほど悪くありません。 ただし、重大な問題が1つあります。Goのエラーは、単に文字列を返す関数（ `Error()` ）を実装するインターフェイスです。 したがって、予想されるエラーコードをコードベースにハードコーディングしていますが、これは理想的ではありません。 このハードコードされた文字列は、マジックストリングと呼ばれます。 そしてその主な問題は柔軟性です：ある時点でエラーを表すために使用される文字列値を変更すると、多くの異なる場所で更新しない限り、コードは（そっと）壊れます。 私たちのコードは密結合されています。それは、その特定のマジックストリングと、コードベースが成長しても変わらないという前提に依存しています。

クライアントが独自のコードでパッケージを使用すると、さらに悪い状況が発生します。 パッケージを更新し、エラーを表す文字列を変更することを決定したことを想像してください。クライアントのソフトウェアが突然壊れます。 これは明らかに避けたいことです。 幸いなことに、修正は非常に簡単です。

```go
package clean

var (
	NullItem = Item{}

	ErrItemNotFound = errors.New("item could not be found in the store") 
)

func (store *Store) GetItem(id string) (Item, error) {
	store.mtx.Lock()
	defer store.mtx.Unlock()

	item, ok := store.items[id]
	if !ok {
		return NullItem, ErrItemNotFound
	}
	return item, nil
}
```

エラーを変数（ `ErrItemNotFound` ）として単純に表すことにより、このパッケージを使用するすべてのユーザーが、返される実際の文字列ではなく変数に対してチェックできることを確認しました:

```go
func GetItemHandler(w http.ReponseWriter, r http.Request) {
	item, err := clean.GetItem("123")
	if err != nil {
		if err == clean.ErrItemNotFound {
			http.Error(w, err.Error(), http.StatusNotFound)
			return
		}
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
	} 
	json.NewEncoder(w).Encode(item)
}
```

これはいい感じで、ずっと安全です。 読みやすいと言う人もいます。 より詳細なエラーメッセージの場合、特定のエラーが返された理由に関するメッセージではなく、単にErrItemNotFoundを読む方が開発者にとって確実に望ましいでしょう。


このアプローチはエラーに限定されず、他の戻り値に使用できます。 例として、以前のように `Item{}` の代わりに `NullItem` を返します。 定義されたオブジェクトを返すときに初期化するのではなく、定義済みのオブジェクトを返すことが望ましいシナリオは数多くあります。

前の例で行ったようにデフォルトの `NullItem` 値を返すことも、特定の場合により安全です。 例として、パッケージのユーザーはエラーのチェックを忘れて、1つ以上のプロパティ値としてデフォルト値 `nil` を含む空の構造体を指す変数を初期化することになります。 コードの後半でこの `nil` 値にアクセスしようとすると、クライアントソフトウェアは `panic` になります。 ただし、代わりにカスタムのデフォルト値を返す場合、そうでなければデフォルトで `nil` になるすべての値が確実に初期化されます。 したがって、ユーザーのソフトウェアに ` panic` を引き起こさないようにします。

これも私たちに利益をもたらします。 考慮してください：デフォルト値を返さずに同じ安全性を実現したい場合、このタイプの空の値を返すすべての場所でコードを変更する必要があります。 ただし、デフォルト値のアプローチでは、コードを1か所で変更するだけで済みます:

```go
var NullItem = Item{
	itemMap: map[string]Item{},
}
```

> 注：多くのシナリオでは、エラーチェックが欠落していることを示すために、実際に `panic` を呼び出すことが推奨されます。

> 注：Goのすべてのインターフェイスプロパティには、 `nil` のデフォルト値があります。 これは、これがインターフェイスプロパティを持つ構造体に役立つことを意味します。 これは、チャネル、マップ、およびスライスを含む構造体にも当てはまります。これらの構造体には、 `nil` 値も含まれる可能性があります。

#### Returning Dynamic Errors

確かに、エラー変数を返すことが実際には実行できないかもしれないいくつかのシナリオがあります。 カスタマイズされたエラーの情報が動的である場合、エラーイベントをより具体的に記述したい場合、静的エラーを定義して返すことはできません。 以下に例を示します:

```go
func (store *Store) GetItem(id string) (Item, error) {
	store.mtx.Lock()
	defer store.mtx.Unlock()

	item, ok := store.items[id]
	if !ok {
		return NullItem, fmt.Errorf("Could not find item with ID: %s", id)
	}
	return item, nil
}
```

ではどうすればいいでしょう？ これらの種類の動的エラーを処理して返すための明確な方法や標準的な方法はありません。 私の個人的な好みは、機能が少し追加された新しいインターフェースを返すことです。

```go
type ErrorDetails interface {
	Error() string
	Type() string
}

type errDetails struct {
	errtype error
	details interface{}
}

func NewErrorDetails(err error, details ...interface{}) ErrorDetails {
	return &errDetails{
		errtype: err,
		details: details,
	}
}

func (err *errDetails) Error() string {
	return fmt.Sprintf("%v: %v", err.errtype, err.details)
}

func (err *errDetails) Type() error {
	return err.errtype
}
```

この新しいデータ構造は、引き続き標準エラーとして機能します。 インターフェース実装であるため、 `nil` と比較できます。また、 `.Error()` を呼び出すことができるため、既存の実装を壊すことはありません。 ただし、現在のエラーには動的な詳細が含まれているにもかかわらず、以前と同じようにエラーの種類を確認できるという利点があります:

```go
func (store *Store) GetItem(id string) (Item, error) {
	store.mtx.Lock()
	defer store.mtx.Unlock()

	item, ok := store.items[id]
	if !ok {
		return NullItem, NewErrorDetails(errors.New("Could not find item with ID"), "1")
	}
	return item, nil
}
```

そして、HTTPハンドラー関数をリファクタリングして、特定のエラーを再度チェックできます。

```go
func GetItemHandler(w http.ReponseWriter, r http.Request) {
	item, err := clean.GetItem("123")
	if err != nil {
		if err.Type() == clean.ErrItemNotFound {
			http.Error(w, err.Error(), http.StatusNotFound)
			return
		}
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
	} 
	json.NewEncoder(w).Encode(item)
}
```

### Nil Values

Goの物議を醸す側面は、 `nil` の追加です。 この値は、Cの `NULL` 値に対応し、基本的には初期化されていないポインターです。  `nil` が引き起こす可能性のある問題のいくつかを既に見てきましたが、要約すると： `nil` 値のメソッドまたはプロパティにアクセスしようとすると壊れます。 したがって、可能な場合は `nil` 値を返さないようにすることをお勧めします。 これにより、コードのユーザーが誤って `nil` 値にアクセスする可能性が低くなります。


他のシナリオでは、不必要な痛みを引き起こす可能性のある `nil` 値を見つけるのが一般的です。 この例は、構造体を誤って初期化することです（下の例のように）。これにより、 `nil` プロパティが含まれる可能性があります。 アクセスすると、これらの `nil` sはpanicを引き起こします。

```go
type App struct {
	Cache *KVCache
}

type KVCache struct {
	mtx sync.RWMutex
	store map[string]string
}

func (cache *KVCache) Add(key, value string) {
	cache.mtx.Lock()
	defer cache.mtx.Unlock()

	cache.store[key] = value
}
```

このコードはまったく問題ありません。 ただし、内部の `Cache` プロパティを初期化せずに、アプリを誤って初期化できる危険があります。 次のコードが呼び出されると、アプリケーションはpanicになります:

```go
app := App{}
app.Cache.Add("panic", "now")
```

`Cache` プロパティは初期化されたことがないため、 `nil` ポインターです。 したがって、ここで行ったように `Add` メソッドを呼び出すと、panicが発生し、次のメッセージが表示されます:

> panic: runtime error: invalid memory address or nil pointer dereference


代わりに、 `App` 構造体の `Cache` プロパティをプライベートプロパティに変更し、それにアクセスするゲッターのようなメソッドを作成できます。 これにより、返されるものをより詳細に制御できます。 具体的には、 `nil` 値を返さないようにします:

```go
type App struct {
	cache *KVCache
  }
  
  func (app *App) Cache() *KVCache {
	if app.cache == nil {
		app.cache = NewKVCache()
	}
	return app.cache
}
```

以前panicに陥っていたコードは、次のようにリファクタリングされます:

```go
app := App{}
app.Cache().Add("panic", "now")
```

これにより、パッケージのユーザーが実装を心配したり、パッケージを安全でない方法で使用しているかどうかを心配したりする必要がなくなります。 心配する必要があるのは、独自のクリーンなコードを書くことだけです。

> 注：同様に安全な結果を達成する他の方法があります。 しかし、これは最も簡単なアプローチだと思います。


### Pointers in Go

Goのポインターは、かなり広範なトピックです。 これらは言語での作業の非常に大きな部分です。そのため、ポインタと言語での動作に関する知識がなければGoを書くことは本質的に不可能です。 したがって、不要な複雑さを追加せずに（そしてコードベースをクリーンに保ちながら）ポインターを使用する方法を理解することが重要です。 Goでのポインターの実装方法の詳細は確認しないことに注意してください。 代わりに、Goポインターの癖とそれらの処理方法に焦点を当てます。

ポインターはコードを複雑にします。 注意を怠り、ポインターを誤って使用すると、デバッグが特に難しい厄介な副作用やバグが発生する可能性があります。 このドキュメントの最初の部分で説明したクリーンなコードを書くという基本原則を守ることで、少なくともコードに不必要な複雑さをもたらす可能性を減らすことができます。

#### Pointer Mutability

グローバル変数または主にスコープされた変数のコンテキストでの可変性の問題をすでに見てきました。 ただし、可変性は必ずしも悪いことではないため、100％純粋な関数型プログラムを作成することを推奨しているわけではありません。 可変性は強力なツールですが、本当に必要な場合にのみ使用すべきです。 理由を示すコード例を見てみましょう:

```go
func (store *UserStore) Insert(user *User) error {
	if store.userExists(user.ID) {
		return ErrItemAlreaydExists
	}
	store.users[user.ID] = user
	return nil
}

func (store *UserStore) userExists(id int64) bool {
	_, ok := store.users[id]
	return ok
}
```

一見、これはそれほど悪くはないようです。 実際、共通のリスト構造に対するかなり単純な挿入関数のように見えることさえあります。 入力としてポインターを受け入れ、この `id` を持つ他のユーザーが存在しない場合、提供されたユーザーポインターをリストに挿入します。 次に、新しいユーザーを作成するためにパブリックAPIでこの機能を使用します:

```go
func CreateUser(w http.ResponseWriter, r *http.Request) {
	user, err := parseUserFromRequest(r)
	if err != nil {
    	http.Error(w, err, http.StatusBadRequest)
    	return
	}
	if err := insertUser(w, user); err != nil {
		http.Error(w, err, http.StatusInternalServerError)
		return
    }
}

func insertUser(w http.ResponseWriter, user User) error {
	if err := store.Insert(user); err != nil {
		return err
	}
	user.Password = ""
	return json.NewEncoder(w).Encode(user)
}
```

もう一度、一見すべてがうまくいっているように見えます。 受信したリクエストからユーザーを解析し、ユーザー構造体をストアに挿入します。 ユーザーをストアに正常に挿入したら、ユーザーをJSONオブジェクトとしてクライアントに返す前に、パスワードを空の文字列に設定します。 通常、パスワードがハッシュ化されたユーザーオブジェクトを返しますがハッシュ化されたパスワードを返したくないため、これは非常に一般的な方法です。

ただし、マップに基づくメモリ内ストアを使用していると想像してください。 このコードは、予期しない結果を生成します。 ユーザーストアを確認すると、HTTPハンドラー関数でユーザーパスワードに加えた変更が、ストア内のオブジェクトにも影響を与えていることがわかります。 これは、 `parseUserFromRequest` によって返されるポインターアドレスが、実際の値ではなく、ストアに入力されたものだからです。 したがって、参照解除されたパスワードの値を変更すると、ストアでポイントしているオブジェクトの値が変更されます。

これは、可変性と変数スコープの両方が、誤って使用されたときにいくつかの重大な問題とバグを引き起こす可能性があることの良い例です。 関数の入力パラメーターとしてポインターを渡すとき、データが指している変数のスコープを拡張しています。 さらに心配なのは、スコープを未定義レベルに拡大しているという事実です。 変数の範囲をほぼグローバルレベルに拡大しています。 上記の例で示されているように、これは特に見つけにくく根絶するのが難しい悲惨なバグにつながる可能性があります。

幸いなことに、これに対する修正はかなり簡単です。

```go
func (store *UserStore) Insert(user User) error {
	if store.userExists(user.ID) {
		return ErrItemAlreaydExists
	}
	store.users[user.ID] = &user
	return nil
}
```

`User` 構造体にポインターを渡す代わりに、 `User` のコピーを渡します。 まだストアへのポインタを保存しています。 ただし、関数の外部からポインターを保存する代わりに、コピーされた値へのポインターを保存します。コピーされた値のスコープは関数内にあります。 これにより当面の問題は修正されますが、注意しないとさらに問題が発生する可能性があります。 次のコードを検討してください:

```go
func (store *UserStore) Get(id int64) (*User, error) {
	user, ok := store.users[id]
	if !ok {
		return EmptyUser, ErrUserNotFound
	}
	return store.users[id], nil
}
```

繰り返しになりますが、これはストアのゲッター関数の非常に標準的な実装です。 ただし、ポインターのスコープを再度拡大しているため、予期しない副作用が発生する可能性があるため、コードは依然として不適切です。 ユーザーストアに格納している実際のポインター値を返すとき、基本的にアプリケーションの他の部分にストア値を変更する機能を与えています。 これは混乱の原因になります。 ストアは、値を変更できる唯一のエンティティでなければなりません。 これに対する最も簡単な修正方法は、ポインターを返すのではなく、ユーザーの値を返すことです。

> 注：アプリケーションが複数のスレッドを使用する場合を考えてください。 このシナリオでは、同じメモリの場所にポインターを渡すと、競合状態になる可能性もあります。 つまり、データが破損する可能性があるだけでなく、データの競合によるpanicを引き起こす可能性もあります。

ポインターを返すことに本質的に問題はないことに注意してください。 ただし、ポインターを使用する場合は、変数のスコープ（およびそれらの変数を指す所有者の数）の拡張が最も重要な考慮事項です。 これは、以前の例をよくない臭いがする操作として分類するものです。 これは、一般的なGoコンストラクターが本当に良い理由でもあります:

```go
func AddName(user *User, name string) {
	user.Name = name
}
```

関数を呼び出した人が定義する変数スコープは、関数が戻った後も同じままなので、これは問題ありません。 関数呼び出し元が変数の唯一の所有者であるという事実と相まって、これはポインターが予期しない方法で操作できないことを意味します。

#### Closures Are Function Pointers

Goでインターフェイスを使用する次のトピックに進む前に、一般的な代替手段を紹介します。 Cプログラマーが「関数ポインター」として知っているものであり、他のほとんどのプログラミング言語がクロージャと呼んでいるものです。 クロージャは、呼び出すことができる関数を表す（指す）ことを除いて、単純な入力パラメータです。 JavaScriptでは、クロージャをコールバックとして使用することは非常に一般的です。クロージャは、非同期操作が完了した後に呼び出される単なる関数です。 Goでは、この概念は実際にはありません。 ただし、クロージャを使用して、別のハードルであるジェネリックの欠如を部分的に克服できます。

次の関数シグネチャを検討してください:

```go
func something(closure func(float64) float64) float64 { ... }
```

ここでは、`something` が入力として別の関数（クロージャ）を受け取り、float64を返します。 入力関数は、入力としてfloat64を受け取り、float64も返します。 このパターンは、疎結合アーキテクチャを作成するのに特に役立ち、コードの他の部分に影響を与えずに機能を簡単に追加できます。 何らかの形式で操作するデータを含む構造体があるとします。 この構造体の `Do()` メソッドを使用して、そのデータに対して操作を実行できます。 前もって操作を知っていれば、明らかに `Do()` メソッドでそのロジックを直接処理できます:

```go
func (datastore *Datastore) Do(operation Operation, data []byte) error {
	switch(operation) {
	case COMPARE:
		return datastore.compare(data)
	case CONCAT:
		return datastore.add(data)
	default:
		return ErrUnknownOperation
	}
}
```

しかし、ご想像のとおり、この関数は非常に厳格であり、 `Datastore` 構造体に含まれるデータに対して所定の操作を実行します。 ある時点でさらに多くの操作を導入したい場合、`Do` メソッドを維持するのが難しいかなり多くの無関係なロジックで肥大化してしまいます。 この関数は、実行する操作を常に気にし、各操作に対して多数のネストされたオプションを循環する必要があります。 また、ほとんどのOOP言語のようにGoで構造メソッドを拡張する方法がないため、パッケージコードを編集するためのアクセス権を持たない `Datastore` オブジェクトを使用したい開発者にとっても問題になる可能性があります。

代わりに、クロージャを使用して別のアプローチを試してみましょう:

```go
func (datastore *Datastore) Do(operation func(data []byte, data []byte) ([]byte, error), data []byte) error {
	result, err := operation(datastore.data, data)
	if err != nil {
		return err
	}
	datastore.data = result
	return nil
}
  
func concat(a []byte, b []byte) ([]byte, error) {
	...
}
  
func main() {
	...
	datastore.Do(concat, data)
	...
}
```

Doの関数シグネチャが非常に乱雑になっていることがすぐにわかります。 また、別の問題もあります。クロージャはジェネリックではありません。 実際に `concat` が入力として2バイト以上の配列を取得できるようにしたいことがわかったらどうなりますか？ または、（`data [] byte、data [] byte`）よりも多いまたは少ない入力値を必要とする可能性がある完全に新しい機能を追加する場合はどうでしょうか？

この問題を解決する1つの方法は、`concat` 関数を変更することです。 以下の例では、入力引数として1バイト配列のみを使用するように変更しましたが、逆の場合も同様です:

```go
func concat(data []byte) func(data []byte) ([]byte, error) {
	return func(concatting []byte) ([]byte, error) {
		return append(data, concatting), nil
	}
}
  
func (datastore *Datastore) Do(operation func(data []byte) ([]byte, error)) error {
	result, err := operation(datastore.data)
	if err != nil {
		return err
	}
	datastore.data = result
	return nil
}
  
func main() {
	...
	datastore.Do(compare(data))
	...
}
```

`Do` メソッドシグネチャから `concat` メソッドシグネチャにいくつかのコードを効果的に移動したことに注目してください。 ここで、concat関数はさらに別の関数を返します。 返された関数内に、concat関数に最初に渡された入力値を保存します。 したがって、返される関数は、単一の入力パラメーターを取ることができます。 関数ロジック内で、元の入力値に追加します。 新しく導入された概念として、これは非常に奇妙に見えるかもしれません。 ただし、これをオプションとして使用することに慣れるのは良いことです。 ロジックカップリングを緩め、肥大化した関数を取り除くのに役立ちます。


次のセクションでは、インターフェイスについて説明します。 それを行う前に、インターフェースとクロージャの違いについて少し話をしましょう。 まず、インターフェイスとクロージャがいくつかの一般的な問題を確実に解決することは注目に値します。 ただし、Goでインターフェイスを実装する方法により、特定の問題に対してインターフェイスを使用するかクロージャを使用するかを決定するのが難しい場合があります。 通常、インターフェースを使用するかクロージャを使用するかは重要ではありません。 適切な選択は、当面の問題を解決する方です。 通常、操作が本質的に単純な場合、クロージャは実装がより簡単になります。 ただし、クロージャに含まれるロジックが複雑になるとすぐに、代わりにインターフェイスの使用を強く検討する必要があります。

Dave Cheneyは、このトピックに関する優れた記事と講演を行っています

- [Do not fear first class functions \| Dave Cheney](https://dave.cheney.net/2016/11/13/do-not-fear-first-class-functions)
- [YouTube](https://www.youtube.com/watch?v=5buaPyJ0XeQ&t=9s)

Jon Bodnerの関連する講演もあります:

- [YouTube](https://www.youtube.com/watch?v=5IKcPMJXkKs)

### Interfaces in Go

一般的に、インターフェイスの処理に対するGoのアプローチは、他の言語のアプローチとはまったく異なります。 インターフェイスは、JavaやC＃のように明示的に実装されていません。 むしろ、インターフェースの契約を満たせば暗黙的に作成されます。 例として、これは、 `Error()` メソッドを持つ構造体が `Error` インターフェイスを実装（または「満たせば」）し、 `error` として返されることを意味します。 このインターフェイスの実装方法は非常に簡単で、Goのテンポの良さ、ダイナミックさを感じられます。

しかし、このアプローチには欠点があります。 インターフェースの実装は明示的ではなくなったため、どのインターフェースが構造体によって実装されているかを確認することは困難です。 したがって、可能な限り少ないメソッドでインターフェイスを定義するのが一般的です。 これにより、特定の構造体がインターフェイスの契約を満たしているかどうかを理解しやすくなります。

別の方法は、具象型ではなくインターフェイスを返すコンストラクタを作成することです：

```go
type Writer interface {
	Write(p []byte) (n int, err error)
}

type NullWriter struct {}

func (writer *NullWriter) Write(data []byte) (n int, err error) {
	// do nothing
	return len(data), nil
}

func NewNullWriter() io.Writer {
	return &NullWriter{}
}
```

上記の関数は、 `NullWriter` 構造体が `Writer` インターフェイスを実装することを保証します。 `NullWriter` から `Write` メソッドを削除すると、コンパイルエラーが発生します。 これは、コードが期待どおりに動作し、無効なコードを記述しようとした場合にセーフティネットとしてコンパイラに依存できるようにするための良い方法です。

場合によっては、コンストラクターを記述することが望ましくない場合があります。または、コンストラクターがインターフェイスではなく具象型を返すようにしたい場合があります。 例として、 `NullWriter` 構造体には初期化時に設定するプロパティがないため、コンストラクターの作成は少し冗長です。 したがって、インターフェイスの互換性を確認するために、より冗長でない方法を使用できます。

```go
type Writer interface {
	Write(p []byte) (n int, err error)
}

type NullWriter struct {}
var _ io.Writer = &NullWriter{}
```

上のコードでは、io.Writerの型割り当てを使用して、Goの `blank identifier` で変数を初期化しています。 これにより、廃棄される前に、 `io.Writer` インターフェイスコントラクトを満たすために変数がチェックされます。 インターフェイスのフルフィルメントをチェックするこの方法により、いくつかのインターフェイス契約が満たされていることをチェックすることもできます。

```go
type NullReaderWriter struct{}
var _ io.Writer = &NullWriter{}
var _ io.Reader = &NullWriter{}
```

上記のコードから、どのインターフェースを満たす必要があるかを非常に簡単に理解できます。 これにより、コンパイラーがコンパイル時に役立つことが保証されます。 したがって、これは一般的に、インターフェース契約の履行をチェックするための推奨されるソリューションです。

与えられた構造体がどのインターフェースを実装するかについて、より明示的にしようとする別の方法があります。 ただし、この3番目の方法は、実際に私たちが望むものの反対になります。 埋め込みインターフェースを構造体プロパティとして使用する必要があります。

> Wait what? – Presumably most people

よくない臭いのGoの禁断の森に深く飛び込む前に、少し巻き戻しましょう。 Goでは、埋め込み構造体を構造体定義の継承の型として使用できます。 再利用可能な構造体を定義することでコードを分離できるため、これは本当に素晴らしいことです。

```go
type Metadata struct {
	CreatedBy types.User
}

type Document struct {
	*Metadata
	Title string
	Body string
}

type AudioFile struct {
	*Metadata
	Title string
	Body string
}
```

上記では、多くの異なる構造型で使用する可能性が高いプロパティフィールドを提供する `Metadata` オブジェクトを定義しています。 構造体でプロパティを直接明示的に定義するのではなく、埋め込まれた構造体を使用することの良い点は、 `Metadata` フィールドを分離したことです。 `Metadata` オブジェクトの更新を選択した場合、1か所で変更できます。 これまで何度か見てきたように、コードの1箇所を変更しても他の部分が壊れないようにする必要があります。 これらのプロパティを一元化することで、 `Metadata` が埋め込まれた構造が同じプロパティを持つことを明確にします。これは、インターフェイスを満たす構造が同じメソッドを持つ方法と同じです。

次に、コンストラクターを使用して、 `Metadata` 構造体に変更を加えたときにコードが破損しないようにする方法の例を見てみましょう。

```go
func NewMetadata(user types.User) Metadata {
	return &Metadata{
		CreatedBy: user,
	}
}

func NewDocument(title string, body string) Document {
	return Document{
		Metadata: NewMetadata(),
		Title: title,
		Body: body,
	}
}
```

後の時点で、 `Metadata` オブジェクトの `CreatedAt` フィールドも必要だと判断したとします。  `NewMetadata` コンストラクタを更新するだけで、これを簡単に実現できます。

```go
func NewMetadata(user types.User) Metadata {
	return &Metadata{
		CreatedBy: user,
		CreatedAt: time.Now(),
	}
}
```

これで、`Document` 構造と `AudioFile` 構造の両方が更新され、これらのフィールドも構築時に設定されます。 これは、デカップリングの背後にある基本原則であり、コードの保守性を確保する優れた例です。 既存のコードを壊さずに新しいメソッドを追加することもできます:

```go
type Metadata struct {
	CreatedBy types.User
	CreatedAt time.Time
	UpdatedBy types.User
	UpdatedAt time.Time
}

func (metadata *Metadata) AddUpdateInfo(user types.User) {
	metadata.UpdatedBy = user
	metadata.UpdatedAt = time.Now()
}
```

繰り返しになりますが、残りのコードベースを壊すことなく、新しい機能を導入することができました。 この種のプログラミングにより、新機能の実装は非常に迅速かつ簡単になります。これは、クリーンなコードを記述することで実現しようとしていることです。

埋め込みインターフェースを使用したインターフェース契約履行のトピックに戻りましょう。 例として次のコードを検討してください。

```go
type NullWriter struct {
	Writer
}

func NewNullWriter() io.Writer {
	return &NullWriter{}
}
```

上記のコードがコンパイルされます。 技術的には、 `NullWriter` はこのインターフェイスに関連付けられているすべての関数を継承するため、 `NullWriter` に `Writer` のインターフェイスを実装しています。 `NullWriter` が `Writer` インターフェイスを実装していることを示す明確な方法としてこれを見る人もいます。 ただし、この手法を使用する場合は注意が必要です。

```go
func main() {
	w := NewNullWriter()

	w.Write([]byte{1, 2, 3})
}
```

前述のように、上記のコードはコンパイルされます。  `NewNullWriter` は `Writer` を返します。 `NullWriter` は組み込みインターフェイスを介して`io.Writer` の契約を満たしているため、コンパイラによるとすべてが順調です。 ただし、上記のコードを実行すると、次の結果になります。

> panic: runtime error: invalid memory address or nil pointer dereference


何が起こったのでしょう？ Goのインターフェースメソッドは、本質的に関数ポインターです。 この場合、実際のメソッド実装ではなく、インターフェイスの関数を指しているため、実際には `nil` ポインターである関数を呼び出そうとしています。 これが発生しないようにするには、 `NulllWriter` に、実際に実装されたメソッドとともに、インターフェイスコントラクトを満たす構造体を提供する必要があります。

```go
func main() {
	w := NullWriter{
		Writer: &bytes.Buffer{},
	}
  
	w.Write([]byte{1, 2, 3})
}
```

> 注：上記の例では、`Writer` は埋め込み `io.Writer` インターフェイスを参照しています。  `w.Writer.Write()` でこのプロパティにアクセスして、 `Write` メソッドを呼び出すこともできます。

panicを引き起こすことはなくなり、 `NullWriter` をWriterとして使用できるようになりました。 前述のように、この初期化プロセスは、 `nil` として初期化されるプロパティを持つことと大差ありません。 したがって、論理的には、同様の方法でそれらを処理しようとする必要があります。 ただし、ここで埋め込みインターフェイスを使用するのが少し難しくなります。 前のセクションで、潜在的な `nil` 値を処理する最良の方法は、問題のプロパティをプライベートにし、パブリックgetterメソッドを作成することであると説明しました。 このようにして、私たちのプロパティが実際に `nil` ではないことを保証できます。 残念ながら、これは本質的に常に公開されているため、これは埋め込みインターフェースでは不可能です。

埋め込みインターフェイスを使用することで発生する別の懸念は、インターフェイスメソッドが部分的に上書きされることによる混乱の可能性です:

```go
type MyReadCloser struct {
	io.ReadCloser
}
  
func (closer *ReadCloser) Read(data []byte) { ... }
  
func main() {
	closer := MyReadCloser{}
	
	closer.Read([]byte{1, 2, 3}) 	// works fine
	closer.Close() 		// causes panic
	closer.ReadCloser.Closer() 		// no panic 
}
```

これは、C#やJavaなどの言語で一般的なメソッドをオーバーライドしているように見えますが、実際にはそうではありません。 Goは継承をサポートしていません（したがって、スーパークラスの概念はありません）。 振る舞いを模倣することはできますが、言語の組み込み部分ではありません。 インターフェースの埋め込みなどのメソッドを注意せずに使用することで、混乱を招く可能性のあるバグのあるコードを作成できます。

> 注: 埋め込みインターフェイスを使用することは、インターフェイスメソッドのサブセットをテストするためのmock構造を作成する良い方法であると主張する人もいます。 基本的に、埋め込みインターフェイスを使用することにより、インターフェイスのすべてのメソッドを実装する必要はありません。 むしろ、テストする少数のメソッドのみを実装することを選択できます。 テスト/モックのコンテキスト内では、この議論を見ることができますが、私はまだこのアプローチのファンではありません。

クリーンコードとインターフェイスの適切な使用法に戻りましょう。 関数のパラメーターおよび戻り値としてのインターフェースの使用について議論する時が来ました。 Goの関数でのインターフェイスの使用に関する最も一般的なことわざは次のとおりです:

> Be conservative in what you do; be liberal in what you accept from others – Jon Postel

> おもしろい事実：このことわざは、実際にはGoとは関係ありません。 これは、TCPネットワーキングプロトコルの初期の仕様から取られています。

つまり、インターフェイスを受け入れて具象型を返す関数を作成する必要があります。 これは一般的に良い習慣であり、モックを使用してテストを行うときに特に役立ちます。 例として、 `Writer` インターフェイスを入力として受け取り、そのインターフェイスの `Write` メソッドを呼び出す関数を作成できます。

```go
type Pipe struct {
	writer io.Writer
	buffer bytes.Buffer
}

func NewPipe(w io.Writer) *Pipe {
	return &Pipe{
		writer: w,
	}
} 

func (pipe *Pipe) Save() error {
	if _, err := pipe.writer.Write(pipe.FlushBuffer()); err != nil {
		return err
	}
	return nil
}
```

アプリケーションの実行中にファイルに書き込みを行うと仮定しますが、この関数を呼び出すすべてのテストのために新しいファイルに書き込みをしたくありません。 基本的には何もしない新しいモックタイプを実装できます。 基本的に、これは基本的な依存関係の注入とモックにすぎませんが、ポイントはGoで非常に簡単に実現できることです。

```go
type NullWriter struct {}

func (w *NullWriter) Write(data []byte) (int, error) {
	return len(data), nil
}

func TestFn(t *testing.T) {
	...
	pipe := NewPipe(NullWriter{})
	...
}
```

> 注：実際には、`Discard` という名前の `ioutil` パッケージに null writerの実装が組み込まれています。


（別のWriterではなく）`NullWriter` を使用して`Pipe` 構造体を構築する場合、 `Save` 関数を呼び出すと何も起こりません。 私たちがしなければならなかったのは、4行のコードを追加することだけでした。 このため、慣用的なGoではインターフェイスをできるだけ小さくすることが推奨されます。これにより、今見たようなパターンを特に簡単に実装できます。 ただし、このインターフェイスの実装には大きな欠点もあります。

### The Empty `interface{}`

他の言語とは異なり、Goにはジェネリックの実装がありません。 1つについて多くの提案がありましたが、Go言語チームによってすべて却下されました。 残念ながら、ジェネリックがないと、開発者は空の `interface{}` を使用するという非常に頻繁な代替手段を見つける必要があります。 このセクションでは、これらのあまりにも創造的な実装が悪い習慣とクリーンでないコードであると考えられる理由を説明します。 空の `interface{}` の適切な使用例と、それを使用してコードを記述する際の落とし穴を回避する方法もあります。

前のセクションで述べたように、Goは、型が特定のインターフェイスを実装するかどうかを、型がそのインターフェイスのメソッドを実装するかどうかを確認することで決定します。 それでは、空のインターフェイスの場合のように、インターフェイスがメソッドを宣言しないとどうなりますか？

```go
type EmptyInterface interface {}
```

上記は組み込み型の `interface{}`と同等です。 これの自然な結果は、引数として任意の型を受け入れる汎用関数を作成できることです。 これは、printヘルパーなどの特定の種類の機能に非常に役立ちます。 興味深いことに、これは実際には `fmt` パッケージから `Println` 関数に任意の型を渡すことを可能にするものです。

```go
func Println(v ...interface{}) {
	...
}
```

この場合、 `Println` は単一の `interface{}` を受け入れるだけではありません。 むしろ、関数は空の `interface{}` を実装する型のスライスを受け入れます。 空の `interface{}` に関連付けられたメソッドがないため、すべての型が受け入れられ、 `Println` に異なる型のスライスをフィードすることさえ可能になります。 これは、文字列変換（渡す文字列と返す文字列の両方）を処理する際の非常に一般的なパターンです。 これの良い例は、 `json` 標準ライブラリパッケージにあります:

```go
func InsertItemHandler(w http.ResponseWriter, r *http.Request) {
	var item Item
	if err := json.NewDecoder(r.Body).Decode(&item); err != nil {
		http.Error(w, err.Error(), http.StatusBadRequest)
		return
	}

	if err := db.InsertItem(item); err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
	}
	w.WriteHeader(http.StatsOK)
}
```

あまり洗練されていないコードはすべて、 `Deocde` 関数に含まれています。 したがって、この機能を使用する開発者は、型のリフレクションや型キャストについて心配する必要はありません。 具体的な型へのポインタを提供することを心配する必要があります。 これは、 `Decode()` 関数が技術的に具象型を返すためです。 HTTPリクエストの本文から入力される `Item` 値を渡します。 これは、 `interface{}` の値を自分で処理する潜在的なリスクに対処する必要がないことを意味します。

ただし、優れたプログラミング手法で空の `interface{}` を使用する場合でも、まだいくつかの問題があります。 `Item` 型とは関係なく、有効なJSONであるJSON文字列を渡した場合、エラーは表示されません。`item` 変数はデフォルト値のままになります。 そのため、リフレクションやキャストエラーについて心配する必要はありませんが、クライアントから送信されたメッセージが有効な `Item` 型であることを確認する必要があります。 残念ながら、このドキュメントを書いている時点では、空の `interface{}` 型を使用せずにこれらの型の汎用デコーダーを実装するための簡単な方法はありません。

この方法で `interface{}` を使用することの問題は、動的に型付けされた言語として、静的に型付けされた言語であるGoを使用することに傾注していることです。 これは、 `interface{}` 型の貧弱な実装を見るとさらに明確になります。 この最も一般的な例は、ある種の汎用ストアまたはリストを実装しようとする開発者のものです。

`interface{}` を使用して任意の型を格納できる汎用HashMapパッケージを実装しようとする例を見てみましょう。

```go
type HashMap struct {
	store map[string]interface{}
}

func (hashmap *HashMap) Insert(key string, value interface{}) {
	hashmap.store[key] = value
}

func (hashmap *HashMap) Get(id string) (interface{}, error) {
	value, ok := hashmap.store[key]
	if !ok {
		return nil, ErrKeyNotFoundInHashMap
	}
	return value
}
```

> 注：簡単にするため、この例ではスレッドセーフにするのを省略しています。

上記の実装パターンは、実際には非常に多くのGoパッケージで使用されていることに注意してください。  `sync.Map` 型の標準ライブラリ同期パッケージでも使用されます。 この実装の問題は何ですか？ それでは、パッケージの使用例を見てみましょう:

```go
func SomeFunction(id string) (Item, error) {
	itemIface, err := hashmap.Get(id)
	if err != nil {
		return EmptyItem, err
	}
	item, ok := itemIface.(Item)
	if !ok {
		return EmptyItem, ErrCastingItem
	}
	return item, nil
}
```

一見、これはうまく見えます。 ただし、現在許可されている別の型をストアに追加すると、問題が発生し始めます。 `Item` 型以外のものを追加することを妨げるものは何もありません。 誰かが `Item` ではなくポインター `*Item` のような他の型をHashMapに追加し始めるとどうなりますか？ 関数はエラーを返す可能性があります。 最悪なことに、これはテストでさえキャッチされないかもしれません。 システムの複雑さによっては、特にデバッグが難しいいくつかのバグが発生する可能性があります。

このタイプのコードは本番環境に使ってはいけません。 忘れない：Goは（まだ）ジェネリックをサポートしていません。 それは、開発者が当分の間受け入れなければならないという事実です。 ジェネリックを使用する場合は、危険なハッキングに頼るのではなく、ジェネリックをサポートする別の言語を使用する必要があります。

それでは、このコードが実稼働に使われるのをどのように防ぐのでしょうか？ 最も簡単な解決策は、 `interface{}` 値を使用する代わりに、具体的な型で関数を記述することです。 もちろん、これは常に最適なアプローチとは限りません。パッケージ内には、実装が自明ではない機能がある可能性があるためです。 したがって、より適切なアプローチは、必要な機能を公開する一方で型の安全性を確保するラッパーを作成することです:

```go
type ItemCache struct {
	kv tinykv.KV
}
  
func (cache *ItemCache) Get(id string) (Item, error) {
	value, ok := cache.kv.Get(id)
	if !ok {
		return EmptyItem, ErrItemNotFound
	}
	return interfaceToItem(value)
}
  
func interfaceToItem(v interface{}) (Item, error) {
	item, ok := v.(Item)
	if !ok {
		return EmptyItem, ErrCouldNotCastItem
	}
	return item, nil
}
  
func (cache *ItemCache) Put(id string, item Item) error {
	return cache.kv.Put(id, item)
}
```

注記：`tinykv.KV` キャッシュの他の機能の実装は、簡潔にするために省略されています。

上記のラッパーにより、実際の型を使用し、 `interface{}` 型を渡さないようになりました。 したがって、誤って間違った値の型をストアに追加することはできなくなり、型のキャストを可能な限り制限しました。 これは、多少手作業であっても、問題を解決する非常に簡単な方法です。

## Summary

まず、この記事を最後まで読んでくれてありがとう！ クリーンなコードと、どのようにコードベースの保守性、読みやすさ、安定性を確保するのに役立つかについての洞察が得られたことを願っています。

取り上げたすべてのトピックを簡単に要約しましょう:

- Functions-関数の名前はそのスコープを反映する必要があります。 関数のスコープが小さいほど、その名前はより具体的になります。 すべての機能が可能な限り少ない行で単一の目的を果たすようにします。 目安として、関数を5〜8行に制限し、2〜3個の引数のみを受け入れます。
- Variables-関数とは異なり、変数はスコープが小さくなるにつれてより一般的な名前を想定する必要があります。 また、意図しない変更を防ぐために、変数のスコープを可能な限り制限することをお勧めします。 同様に、変数の変更は最小限に抑える必要があります。 これは、変数のスコープが大きくなるにつれて特に重要な考慮事項になります。
- Return Values-可能な場合はいつでも、具象型を返す必要があります。 パッケージのユーザーが間違いを犯すことをできるだけ難しくし、関数が返す値を理解しやすくすること。
- Pointers-ポインタは慎重に使用し、スコープと可変性を最小限に制限してください。 要確認：ガベージコレクションはメモリ管理のみを支援します。 ポインターに関連する他のすべての複雑さを支援するものではありません。
- Interfaces—できるだけインターフェースを使用して、コードの結合を緩めます。 空の `interfaces{}` を使用して、エンドユーザーから可能な限りコードを非表示にして、公開されないようにします。

最後の注意点として、クリーンなコードの概念は特に主観的であり、おそらく変わらないことを言及する価値があります。 しかし、 `gofmt` に関する私の声明と同様に、誰もが同意するものよりも共通の標準を見つけることが重要だと思います。 後者を達成することは非常に困難です。

また、クリーンなコードでは、狂信は決して目標ではないことを理解することも重要です。 ほとんどの場合、コードベースは完全に「クリーン」になることはありません。これは、オフィスのデスクもおそらくそうではないからです。 確かに、この記事で説明されているルールと境界の外に出る余地があります。 ただし、クリーンなコードを記述する最も重要な理由は、自分自身や他の開発者を支援することであることを忘れないでください。 私たちは、生産するソフトウェアの安定性を確保し、障害のあるコードのデバッグを容易にすることにより、エンジニアをサポートしています。 コードを読みやすく、簡単に消化できるようにすることで、仲間の開発者を支援します。 現在のプラットフォームを壊すことなく新しい機能をすばやく導入できる柔軟なコードベースを確立することで、プロジェクトに関わるすべての人を支援します。 ゆっくり行くことで素早く動き、そしてみんなが満足します。

このディスカッションに参加して、Goコミュニティがクリーンコードの概念を定義（および改良）するのを支援してください。 ソフトウェアを改善できるように、自分自身だけでなく、すべての人のために共通の基盤を確立しましょう。
