<!DOCTYPE html>
<html>

<head>
  <meta charset='utf-8'>
  <meta http-equiv='X-UA-Compatible' content='IE=edge'>
  <title>convert WebSafe</title>
  <meta name='viewport' content='width=device-width, initial-scale=1'>
  <script src="https://use.fontawesome.com/releases/v5.3.1/js/all.js"></script>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bulma@0.9.0/css/bulma.min.css" />
  <script src="https://cdn.jsdelivr.net/npm/vue@2.6.11"></script>
  <style>
    html, html * {font-family: 'Miryo UI'}
    .textarea { height: 250px; background-color:#fefefe; }
  </style>
</head>

<body>
  <div id="app" class="columns">
    <div class="column is-10 is-offset-1">
      <h3 class="title">入力</h3>
      <textarea class="textarea mb-2" v-model="txt"></textarea>
      <h3 class="title">出力</h3>
      <textarea class="textarea mb-2" readonly>{{ convert }}</textarea>
    </div>
  </div>

  <script>
    /***************************************************************************************************************************
    機能：文字列変換関数
    引数：ConvCls 変換内容(Gloval変数)
    注意事項：再帰呼び出し　/　javascriptのStringオブジェクトにprototypeで追加　
    ***************************************************************************************************************************/
    var strUpper = 1; //大文字
    var strLower = 2; //小文字
    var strFull = 4; //全角
    var strHalf = 8; //半角
    var strKata = 16; //全角カタカナ
    var strHira = 32; //ひらがな
    var strWebsafe = 64; //半角カタカナ→全角カタカナ

    String.prototype.convert = function (ConvCls, notConv) {
      var str = this,
        ConvFrom, ConvTo;
      //数字定数
      var FullNumber = ["０", "１", "２", "３", "４", "５", "６", "７", "８", "９"];
      var HalfNumber = ["0", "1", "2", "3", "4", "5", "6", "7", "8", "9"];
      //記号定数
      var FullSymbol = ["，", "　", "！", "＃", "＄", "％", "＆", "’", "（", "）", "＊", "＋", "－", "．", "／", "：", "；", "＜", "＝",
        "＞", "？", "＠", "［", "￥", "］", "＾", "＿", "‘", "｛", "｜", "｝", "～", "”", "×"
      ];
      var HalfSymbol = [",", " ", "!", "#", "$", "%", "&", "'", "(", ")", "*", "+", "-", ".", "/", ":", ";", "<", "=",
        ">", "?", "@", "[", "\\", "]", "^", "_", "`", "{", "|", "}", "~", "\"", "*"
      ];
      //アルファベット定数
      var FullUpper = ["Ａ", "Ｂ", "Ｃ", "Ｄ", "Ｅ", "Ｆ", "Ｇ", "Ｈ", "Ｉ", "Ｊ", "Ｋ", "Ｌ", "Ｍ", "Ｎ", "Ｏ", "Ｐ", "Ｑ", "Ｒ", "Ｓ",
        "Ｔ", "Ｕ", "Ｖ", "Ｗ", "Ｘ", "Ｙ", "Ｚ"
      ];
      var HalfUpper = ["A", "B", "C", "D", "E", "F", "G", "H", "I", "J", "K", "L", "M", "N", "O", "P", "Q", "R", "S",
        "T", "U", "V", "W", "X", "Y", "Z"
      ];
      var FullLower = ["ａ", "ｂ", "ｃ", "ｄ", "ｅ", "ｆ", "ｇ", "ｈ", "ｉ", "ｊ", "ｋ", "ｌ", "ｍ", "ｎ", "ｏ", "ｐ", "ｑ", "ｒ", "ｓ",
        "ｔ", "ｕ", "ｖ", "ｗ", "ｘ", "ｙ", "ｚ"
      ];
      var HalfLower = ["a", "b", "c", "d", "e", "f", "g", "h", "i", "j", "k", "l", "m", "n", "o", "p", "q", "r", "s",
        "t", "u", "v", "w", "x", "y", "z"
      ];
      //カタカナ定数	「ゐ・ヰ」「ゑ・ヱ」は対象外
      var FullKata = ["ガ", "ギ", "グ", "ゲ", "ゴ", "ザ", "ジ", "ズ", "ゼ", "ゾ", "ダ", "ヂ", "ヅ", "デ", "ド", "バ", "ビ", "ブ", "ベ",
        "ボ",
        "パ", "ピ", "プ", "ペ", "ポ", "ヴ", "ヲ", "ァ", "ィ", "ゥ", "ェ", "ォ", "ャ", "ュ", "ョ", "ッ",
        "ア", "イ", "ウ", "エ", "オ", "カ", "キ", "ク", "ケ", "コ", "サ", "シ", "ス", "セ", "ソ", "タ", "チ", "ツ", "テ", "ト",
        "ナ", "ニ", "ヌ", "ネ", "ノ", "ハ", "ヒ", "フ", "ヘ", "ホ", "マ", "ミ", "ム", "メ", "モ", "ヤ", "ユ", "ヨ",
        "ラ", "リ", "ル", "レ", "ロ", "ワ", "ン", "。", "「", "」", "、", "・", "ー"
      ];
      var HalfKata = ["ｶﾞ", "ｷﾞ", "ｸﾞ", "ｹﾞ", "ｺﾞ", "ｻﾞ", "ｼﾞ", "ｽﾞ", "ｾﾞ", "ｿﾞ", "ﾀﾞ", "ﾁﾞ", "ﾂﾞ", "ﾃﾞ", "ﾄﾞ", "ﾊﾞ",
        "ﾋﾞ", "ﾌﾞ", "ﾍﾞ", "ﾎﾞ",
        "ﾊﾟ", "ﾋﾟ", "ﾌﾟ", "ﾍﾟ", "ﾎﾟ", "ｳﾞ", "ｦ", "ｧ", "ｨ", "ｩ", "ｪ", "ｫ", "ｬ", "ｭ", "ｮ", "ｯ",
        "ｱ", "ｲ", "ｳ", "ｴ", "ｵ", "ｶ", "ｷ", "ｸ", "ｹ", "ｺ", "ｻ", "ｼ", "ｽ", "ｾ", "ｿ", "ﾀ", "ﾁ", "ﾂ", "ﾃ", "ﾄ",
        "ﾅ", "ﾆ", "ﾇ", "ﾈ", "ﾉ", "ﾊ", "ﾋ", "ﾌ", "ﾍ", "ﾎ", "ﾏ", "ﾐ", "ﾑ", "ﾒ", "ﾓ", "ﾔ", "ﾕ", "ﾖ",
        "ﾗ", "ﾘ", "ﾙ", "ﾚ", "ﾛ", "ﾜ", "ﾝ", "｡", "｢", "｣", "､", "･", "ｰ"
      ];
      //ひらがな定数	「ゐ・ヰ」「ゑ・ヱ」は対象外
      var HiraKana = ["が", "ぎ", "ぐ", "げ", "ご", "ざ", "じ", "ず", "ぜ", "ぞ", "だ", "ぢ", "づ", "で", "ど", "ば", "び", "ぶ", "べ",
        "ぼ",
        "ぱ", "ぴ", "ぷ", "ぺ", "ぽ", "う゛", "を", "ぁ", "ぃ", "ぅ", "ぇ", "ぉ", "ゃ", "ゅ", "ょ", "っ",
        "あ", "い", "う", "え", "お", "か", "き", "く", "け", "こ", "さ", "し", "す", "せ", "そ", "た", "ち", "つ", "て", "と",
        "な", "に", "ぬ", "ね", "の", "は", "ひ", "ふ", "へ", "ほ", "ま", "み", "む", "め", "も", "や", "ゆ", "よ",
        "ら", "り", "る", "れ", "ろ", "わ", "ん", "。", "「", "」", "、", "・", "ー"
      ];

      switch (ConvCls) {
        //********個別変換*******
        case 1: //strUpper
          return str.toUpperCase();
        case 2: //strLower
          return str.toLowerCase();
        case 4: //strFull
          ConvFrom = HalfNumber.concat(HalfLower, HalfUpper, HalfKata, HalfSymbol);
          ConvTo = FullNumber.concat(FullLower, FullUpper, FullKata, FullSymbol);
          break;
        case 8: //strHalf
          ConvFrom = FullNumber.concat(FullLower, FullUpper, FullKata, FullSymbol);
          ConvTo = HalfNumber.concat(HalfLower, HalfUpper, HalfKata, HalfSymbol);
          break;
        case 16: //strKata
          ConvFrom = HiraKana.concat(HalfKata);
          ConvTo = FullKata.concat(FullKata);
          break;
        case 32: //strHira
          ConvFrom = FullKata.concat(HalfKata);
          ConvTo = HiraKana.concat(HiraKana);
          break;
        case 64: //strWebsafe
          ConvFrom = HalfKata;
          ConvTo = FullKata;
          break;

          //********組み合わせ変換******* 
        case 5:
          return str.convert(strUpper, notConv).convert(strFull, notConv);
        case 9:
          return str.convert(strUpper, notConv).convert(strHalf, notConv);
        case 17:
          return str.convert(strKata, notConv).convert(strUpper, notConv);
        case 33:
          return str.convert(strHira, notConv).convert(strUpper, notConv);
        case 6:
          return str.convert(strLower, notConv).convert(strFull, notConv);
        case 10:
          return str.convert(strLower, notConv).convert(strHalf, notConv);
        case 18:
          return str.convert(strKata, notConv).convert(strLower, notConv);
        case 34:
          return str.convert(strHira, notConv).convert(strLower, notConv);
        case 20:
          return str.convert(strKata, notConv).convert(strFull, notConv);
        case 36:
          return str.convert(strFull, notConv).convert(strHira, notConv);
        case 40:
          return str.convert(strHira, notConv).convert(strHalf, notConv);
        case 24:
          return str.convert(strKata, notConv).convert(strHalf, notConv);
        case 21:
          return str.convert(strKata, notConv).convert(strUpper, notConv).convert(strFull, notConv);
        case 37:
          return str.convert(strUpper, notConv).convert(strFull, notConv).convert(strHira, notConv);
        case 25:
          return str.convert(strKata, notConv).convert(strUpper, notConv).convert(strHalf, notConv);
        case 41:
          return str.convert(strHira, notConv).convert(strUpper, notConv).convert(strHalf, notConv);
        case 22:
          return str.convert(strKata, notConv).convert(strLower, notConv).convert(strFull, notConv);
        case 38:
          return str.convert(strLower, notConv).convert(strFull, notConv).convert(strHira, notConv);
        case 26:
          return str.convert(strKata, notConv).convert(strLower, notConv).convert(strHalf, notConv);
        case 42:
          return str.convert(strHira, notConv).convert(strLower, notConv).convert(strHalf, notConv);
        case 65:
          return str.convert(strUpper, notConv).convert(strWebsafe, notConv);
        case 66:
          return str.convert(strLower, notConv).convert(strWebsafe, notConv);
        case 72:
          return str.convert(strHalf, notConv).convert(strWebsafe, notConv);
        case 73:
          return str.convert(strHalf, notConv).convert(strUpper, notConv).convert(strWebsafe, notConv);
        case 74:
          return str.convert(strHalf, notConv).convert(strLower, notConv).convert(strWebsafe, notConv);
        default:
          throw new Error("String.convert: 引数の組み合わせが不正です");
      }

      //***********変換実処理**************
      for (var i = 0, len = ConvFrom.length; i < len; i++) {
        var hit = false;
        if (notConv != undefined) {
          for (var j = 0; j < notConv.length; j++) {
            if (ConvFrom[i] == notConv[j]) {
              hit = true;
              break;
            }
          }
        }
        if (!hit) {
          var reg = (".^$[]*+?|()\\".indexOf(ConvFrom[i]) == -1) ?
            new RegExp(ConvFrom[i], "gm") :
            new RegExp("\\" + ConvFrom[i], "gm");
          str = str.replace(reg, ConvTo[i]);
        }
      }
      return str;
    }
  </script>
  <script>
    var app = new Vue({
      el: '#app',
      data: {
        txt: ''
      },
      computed: {
        convert: function(){
          return this.txt.convert(strHalf + strWebsafe);
        }
      }
    });
  </script>
</body>

</html>
