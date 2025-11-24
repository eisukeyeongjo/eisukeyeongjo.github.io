# 13. 僕とお喋りしませんか？

とても基本的な話かもしれないですが、RSpecを記述する際に意識すべきことを備忘録として残します。

- [rspec-coreのREADMEの一文]()
- [どんな会話？]()
- [誰と誰の会話？]()
- [RSpecはただ書くだけだと勿体ない]()
- [具体例]()

### rspec-coreのREADMEの一文

先日チームのメンバーとrspec-coreのリポジトリを読んでいたのですが、READMEに興味深い一文がりました。


その文章がこちら。

> Basic Structure
> RSpec uses the words "describe" and "it" so we can express concepts like a conversation:

参考：[GitHub - rspec/rspec-core: RSpec runner and formatters](https://github.com/rspec/rspec-core)

訳すとこんな感じ？

```
基本構造
RSpecはDescribeとItという単語を使用することで会話のように概念を表現することができるようになります。
```

この一文の「会話のように概念を表現」の部分が特に印象に残りました。

### どんな会話？

会話の具体例が書いてありました。

> "Describe an order."
> "It sums the prices of its line items."

訳すとこんな感じ？

```
orderがどう振る舞うか説明して！
いいよ！これはアイテムの金額を合計するんだ！
```

ちょっと訳が変かもしれませんが、そこにはプログラムがどう振る舞うかという説明が会話形式で記されています。

上記は `describe` と `it` だけのパターンですが、これに `context` を加えると状況（条件）がわかるようになります。

ちなみに `context` は `describe` のaliasなので中身は一緒で用途が違うだけです。
同じ振る舞いをするaliasを用意している点からもRSpecがこの形式を重視しているように思います。

### 誰と誰の会話？

会話とはいったものの、これは誰と誰の会話なのでしょうか。

私は 現在の実装者と未来の実装者（同一人物の場合もある）がRSpecを介して会話して、現在の実装者がプログラムの振る舞いを説明していると解釈すると平和なのかなと考えてます。

もちろんRSpecはあくまでプログラムが期待通りに動作するとこを担保するためのものなので、説明形式でテストケースを作れば良いというわけではありません。

ただこの説明の形式を意識することでより大きな効果を得ることができると考えています。

### RSpecはただ書くだけだと勿体ない

時々引数で文字列を渡していないエグザンプルや適当な文字列を渡しているコンテクストを見かけることがあります。
RSpecは、適切な構造・適切な説明を意識して記述することでプログラムがどのように振る舞うかを教えてくれるので、これらの点を手を抜いてしまうのは勿体ないと思います。

更にRSpec実行時に `--format documentation` のオプションを与えるとドキュメント形式で簡単な仕様がわかるような出力がされます。

私は最近忘れっぽく、先週書いたコードが全く見に覚えのないものだったりすることがあり、、、
このような細かい部分を意識することでRSpecの効果を最大限発揮してバグを減らし質の高いコーディングを心がけようと思いました。

### 具体例

具体例として自分でコードを書いてみました。

```
~/repos $ mkdir rspec-sample
~/repos $ cd rspec-sample/
~/repos/rspec-sample $ bundle init
Writing new Gemfile to /home/ezquerro/repos/rspec-sample/Gemfile
~/repos/rspec-sample $ vi Gemfile 
~/repos/rspec-sample $ cat Gemfile 
# frozen_string_literal: true

source "https://rubygems.org"

# gem "rails"
gem 'rspec-core'

~/repos/rspec-sample $ bundle install --path vendor/bundle
~/repos/rspec-sample $ ls -al
total 51
drwxrwxr-x 4 ezquerro ezquerro   7 Sep  9 14:17 .
drwxrwxr-x 9 ezquerro ezquerro   9 Sep  7 18:41 ..
drwxrwxr-x 2 ezquerro ezquerro   3 Sep  7 18:43 .bundle
-rw-rw-r-- 1 ezquerro ezquerro  94 Sep  7 19:01 Gemfile
-rw-rw-r-- 1 ezquerro ezquerro 205 Sep  7 19:01 Gemfile.lock
drwxrwxr-x 3 ezquerro ezquerro   3 Sep  7 18:43 vendor


~/repos/rspec-sample $ vi tax_calculator.rb 
~/repos/rspec-sample $ cat -n tax_calculator.rb
     1  class TaxCalculator
     2    class PriceMustBeIntegerError < StandardError; end
     3    class TaxRateMustBeFloatError < StandardError; end
     4  
     5    def initialize(original_price, tax_rate)
     6      raise PriceMustBeIntegerError unless original_price.kind_of?(Integer)
     7      raise TaxRateMustBeFloatError unless tax_rate.kind_of?(Float)
     8  
     9      @original_price = Integer(original_price)
    10      @tax_rate       = Float(tax_rate)
    11    end
    12  
    13    def calculate
    14      Integer @original_price * (1 + @tax_rate)
    15    end
    16  end
~/repos/rspec-sample $ irb
irb(main):001:0> require '~/repos/rspec-sample/tax_calculator'
=> true
irb(main):002:0> tc=TaxCalculator.new(1000, 0.13)
=> #<TaxCalculator:0x00007faea664fcd8 @original_price=1000, @tax_rate=0.13>
irb(main):003:0> tc.calculate
=> 1130
```

上記は税抜価格と税率から、税金適用後の価格を算出するクラスのコードです。

そして以下はそのRSpecのコードです。

```
~/repos/rspec-sample $ cat -n tax_calculator_spec.rb
     1  require 'rspec/core'
     2  require '~/repos/rspec-sample/tax_calculator'
     3                   
     4  RSpec.describe TaxCalculator do
     5    let(:price) { rand 1000..9999 } 
     6    let(:tax_rate) { rand 0.01..0.99 }
     7  
     8    describe '#initialize' do
     9      subject { TaxCalculator.new(price, tax_rate) }
    10  
    11      context 'with an Integer object as price and a Float object as tax rate' do
    12        it 'does not raise exception' do
    13          expect{ subject }.not_to raise_error
    14        end
    15  
    16        it 'returns an object of TaxCalculator class' do
    17          is_expected.to be_a_kind_of TaxCalculator
    18        end
    19      end
    20  
    21  
    22      context 'with nil as price' do
    23        let(:price) { nil }
    24  
    25        it 'raises TaxCalculator::PriceMustBeIntegerError' do
    26          expect{ subject }.to raise_error TaxCalculator::PriceMustBeIntegerError
    27        end
    28      end
    29  
    30      context 'with a String object as price' do
    31        let(:price) { '1000' }
    32  
    33        it 'raises TaxCalculator::PriceMustBeIntegerError' do
    34          expect{ subject }.to raise_error TaxCalculator::PriceMustBeIntegerError
    35        end
    36      end
    37  
    38      context 'with a Float object as price' do
    39        let(:price) { 1000.0 }
    40  
    41        it 'raises TaxCalculator::PriceMustBeIntegerError' do
    42          expect{ subject }.to raise_error TaxCalculator::PriceMustBeIntegerError
    43        end
    44      end
    45      
    46      context 'with nil as tax rate' do
    47        let(:tax_rate) { nil }
    48  
    49        it 'raises TaxCalculator::TaxRateMustBeFloatError' do
    50          expect{ subject }.to raise_error TaxCalculator::TaxRateMustBeFloatError
    51        end
    52      end
    53  
    54      context 'with a String object as tax rate' do
    55        let(:tax_rate) { '0.13' }
    56  
    57        it 'raises TaxCalculator::TaxRateMustBeFloatError' do 
    58          expect{ subject }.to raise_error TaxCalculator::TaxRateMustBeFloatError
    59        end
    60      end
    61  
    62      context 'with an Integer object as tax rate' do
    63        let(:tax_rate) { 1 }
    64  
    65        it 'raises TaxCalculator::TaxRateMustBeFloatError' do 
    66          expect{ subject }.to raise_error TaxCalculator::TaxRateMustBeFloatError
    67        end
    68      end
    69    end
    70  
    71    describe '#calculate' do
    72      let(:tax_calculator) { TaxCalculator.new(price, tax_rate) }
    73      subject { tax_calculator.calculate }
    74  
    75      it 'returns the amount after taking taxes into account' do
    76        is_expected.to eq Integer price * (1 + tax_rate)
    77      end
    78    end
    79  end
    80
```

これを会話形式で表現すると以下のようになります。（超訳）

```
太郎「 TaxCalculator#initialize の振る舞いを説明して！」
花子「いいわよ！」
花子「価格として Integer クラスのオブジェクトを、税率として Float クラスのオブジェクトを引数に渡すと例外を発生させず TaxCalculator クラスのオブジェクトを返すよ」
花子「それ以外のクラスのオブジェクトを渡すとそれぞれ TaxCalculator::PriceMustBeIntegerError と TaxCalculator::TaxRateMustBeFloatError が発生するよ」
太郎「ありがとう！ TaxCalculator#calculate も教えて！」
花子「このメソッドを呼び出すと税率が適用された金額を Integer クラスのオブジェクトで返すよ」
太郎「ありがとう！」
```

こんな感じでしょうか？

また、オプションを渡して実行すると仕様が理解できるドキュメント形式で出力されます。

```
~/repos/rspec-sample $ bundle exec rspec tax_calculator_spec.rb --format documentation

TaxCalculator
  #initialize
    with an Integer object as price and a Float object as tax rate
      does not raise exception
      returns an object of TaxCalculator class
    with nil as price
      raises TaxCalculator::PriceMustBeIntegerError
    with a String object as price
      raises TaxCalculator::PriceMustBeIntegerError
    with a Float object as price
      raises TaxCalculator::PriceMustBeIntegerError
    with nil as tax rate
      raises TaxCalculator::TaxRateMustBeFloatError
    with a String object as tax rate
      raises TaxCalculator::TaxRateMustBeFloatError
    with an Integer object as tax rate
      raises TaxCalculator::TaxRateMustBeFloatError
  #calculate
    returns the amount after taking taxes into account

Finished in 0.00506 seconds (files took 0.0943 seconds to load)
9 examples, 0 failures
```

