<template lang="pug">
  div.hello
    h1 Spot The Homograph
    div.controls
      mu-raised-button(label="Instructions", @click="openDialog")
      mu-raised-button(label="Get Homograph", @click="openHomograph")
    mu-dialog(:open="getHomographDialog", title="Game Settings", @close="close")
      ui-select(:has-search='true', label='What is your Native Language?', :options="languages", v-model="chosenLanguage")
      | Difficulty Level {{(difficulty * 100) - 84}}
      mu-slider(name="Difficulty", :max="0.99", :min="0.85", :step="0.01", v-model="difficulty")
      | Frequency {{Math.floor(frequency * 100)}}%
      mu-slider(name="Frequency", :min="0.1", :max="0.5", :step="0.01", v-model="frequency")
      mu-flat-button(slot="actions", secondary, @click="close", label="Cancel")
      mu-flat-button(slot="actions", primary, @click="getUser", label="Get Homograph")
    mu-dialog(:open="dialog", title="Spot The Homograph", @close="close")
      p
        | A Homograph is observed when each of two or more words are spelled the same but not
        | necessarily pronounced the same and having different meanings and origins.
      p
        | In our case, we're looking at words that look like they have the correct spelling,
        | but actually don't. These are words that have characters that look similar to the
        | the correct letter but actually is using a letter that a computer knows is different.
      p
        b
          | This is an experiment to see if you can spot the homograph. You will be presented
          | with a string of text that will contain homographs. To spot them all just tap on the
          | word you think looks a little off.
      mu-flat-button(slot="actions", primary, @click="close", label="OK")
    div.sequence(v-if="homograph.length")
      h2 Make your choices
      mu-flat-button.homograph(v-for="(word, index) in homograph", :label="word", @click="homographClick(index, $event)")
      h3 There are {{homographData.length}} words to find.

    div.selections(v-if="homograph.length")
      h2 Your selections
      mu-flat-button.homograph(v-for="index in selected", :label="homograph[index]")
    div.checkGame(v-if="matches.length")
      h2 Matches
      h3 {{homographData.length - matches.length}} left
      match(v-for="match in matches", :letter-index="match.index", :word="match.word")
    mu-dialog(:open="homographData.length - matches.length === 0 && homographData.length !== 0", title="Great Job!", @close="clear")
      | You managed to find all the Homographs for this problem!
      mu-flat-button(slot="actions", primary, @click="clear", label="Thanks!")

</template>

<script>
  import _ from 'lodash'
  import MuRaisedButton from '../../node_modules/muse-ui/src/raisedButton/raisedButton'
  import MuFlatButton from '../../node_modules/muse-ui/src/flatButton/flatButton'
  import Match from './Matches.vue'
  import MuDialog from '../../node_modules/muse-ui/src/dialog/dialog'
  import UiSelect from '../../node_modules/keen-ui/src/UiSelect'
  /**
   * Created by Matthew Ary on 6/29/17.
   */

  export default {
    components: {
      UiSelect,
      MuDialog,
      Match,
      MuFlatButton,
      MuRaisedButton
    },
    name: 'main',
    data () {
      return {
        userId: null,
        token: null,
        time: 0,
        difficulty: 0.85,
        frequency: 0.10,
        dialog: false,
        getHomographDialog: false,
        homograph: [],
        originalText: [],
        homographData: [],
        selected: [],
        matches: [],
        languages: [
          {'value': 'ab', 'label': 'Abkhaz', 'nativeName': 'аҧсуа'},
          {'value': 'aa', 'label': 'Afar', 'nativeName': 'Afaraf'},
          {'value': 'af', 'label': 'Afrikaans', 'nativeName': 'Afrikaans'},
          {'value': 'ak', 'label': 'Akan', 'nativeName': 'Akan'},
          {'value': 'sq', 'label': 'Albanian', 'nativeName': 'Shqip'},
          {'value': 'am', 'label': 'Amharic', 'nativeName': 'አማርኛ'},
          {'value': 'ar', 'label': 'Arabic', 'nativeName': 'العربية'},
          {'value': 'an', 'label': 'Aragonese', 'nativeName': 'Aragonés'},
          {'value': 'hy', 'label': 'Armenian', 'nativeName': 'Հայերեն'},
          {'value': 'as', 'label': 'Assamese', 'nativeName': 'অসমীয়া'},
          {'value': 'av', 'label': 'Avaric', 'nativeName': 'авар мацӀ, магӀарул мацӀ'},
          {'value': 'ae', 'label': 'Avestan', 'nativeName': 'avesta'},
          {'value': 'ay', 'label': 'Aymara', 'nativeName': 'aymar aru'},
          {'value': 'az', 'label': 'Azerbaijani', 'nativeName': 'azərbaycan dili'},
          {'value': 'bm', 'label': 'Bambara', 'nativeName': 'bamanankan'},
          {'value': 'ba', 'label': 'Bashkir', 'nativeName': 'башҡорт теле'},
          {'value': 'eu', 'label': 'Basque', 'nativeName': 'euskara, euskera'},
          {'value': 'be', 'label': 'Belarusian', 'nativeName': 'Беларуская'},
          {'value': 'bn', 'label': 'Bengali', 'nativeName': 'বাংলা'},
          {'value': 'bh', 'label': 'Bihari', 'nativeName': 'भोजपुरी'},
          {'value': 'bi', 'label': 'Bislama', 'nativeName': 'Bislama'},
          {'value': 'bs', 'label': 'Bosnian', 'nativeName': 'bosanski jezik'},
          {'value': 'br', 'label': 'Breton', 'nativeName': 'brezhoneg'},
          {'value': 'bg', 'label': 'Bulgarian', 'nativeName': 'български език'},
          {'value': 'my', 'label': 'Burmese', 'nativeName': 'ဗမာစာ'},
          {'value': 'ca', 'label': 'Catalan; Valencian', 'nativeName': 'Català'},
          {'value': 'ch', 'label': 'Chamorro', 'nativeName': 'Chamoru'},
          {'value': 'ce', 'label': 'Chechen', 'nativeName': 'нохчийн мотт'},
          {'value': 'ny', 'label': 'Chichewa; Chewa; Nyanja', 'nativeName': 'chiCheŵa, chinyanja'},
          {'value': 'zh', 'label': 'Chinese', 'nativeName': '中文 (Zhōngwén), 汉语, 漢語'},
          {'value': 'cv', 'label': 'Chuvash', 'nativeName': 'чӑваш чӗлхи'},
          {'value': 'kw', 'label': 'Cornish', 'nativeName': 'Kernewek'},
          {'value': 'co', 'label': 'Corsican', 'nativeName': 'corsu, lingua corsa'},
          {'value': 'cr', 'label': 'Cree', 'nativeName': 'ᓀᐦᐃᔭᐍᐏᐣ'},
          {'value': 'hr', 'label': 'Croatian', 'nativeName': 'hrvatski'},
          {'value': 'cs', 'label': 'Czech', 'nativeName': 'česky, čeština'},
          {'value': 'da', 'label': 'Danish', 'nativeName': 'dansk'},
          {'value': 'dv', 'label': 'Divehi; Dhivehi; Maldivian;', 'nativeName': 'ދިވެހި'},
          {'value': 'nl', 'label': 'Dutch', 'nativeName': 'Nederlands, Vlaams'},
          {'value': 'en', 'label': 'English', 'nativeName': 'English'},
          {'value': 'eo', 'label': 'Esperanto', 'nativeName': 'Esperanto'},
          {'value': 'et', 'label': 'Estonian', 'nativeName': 'eesti, eesti keel'},
          {'value': 'ee', 'label': 'Ewe', 'nativeName': 'Eʋegbe'},
          {'value': 'fo', 'label': 'Faroese', 'nativeName': 'føroyskt'},
          {'value': 'fj', 'label': 'Fijian', 'nativeName': 'vosa Vakaviti'},
          {'value': 'fi', 'label': 'Finnish', 'nativeName': 'suomi, suomen kieli'},
          {'value': 'fr', 'label': 'French', 'nativeName': 'français, langue française'},
          {'value': 'ff', 'label': 'Fula; Fulah; Pulaar; Pular', 'nativeName': 'Fulfulde, Pulaar, Pular'},
          {'value': 'gl', 'label': 'Galician', 'nativeName': 'Galego'},
          {'value': 'ka', 'label': 'Georgian', 'nativeName': 'ქართული'},
          {'value': 'de', 'label': 'German', 'nativeName': 'Deutsch'},
          {'value': 'el', 'label': 'Greek, Modern', 'nativeName': 'Ελληνικά'},
          {'value': 'gn', 'label': 'Guaraní', 'nativeName': 'Avañeẽ'},
          {'value': 'gu', 'label': 'Gujarati', 'nativeName': 'ગુજરાતી'},
          {'value': 'ht', 'label': 'Haitian; Haitian Creole', 'nativeName': 'Kreyòl ayisyen'},
          {'value': 'ha', 'label': 'Hausa', 'nativeName': 'Hausa, هَوُسَ'},
          {'value': 'he', 'label': 'Hebrew (modern)', 'nativeName': 'עברית'},
          {'value': 'hz', 'label': 'Herero', 'nativeName': 'Otjiherero'},
          {'value': 'hi', 'label': 'Hindi', 'nativeName': 'हिन्दी, हिंदी'},
          {'value': 'ho', 'label': 'Hiri Motu', 'nativeName': 'Hiri Motu'},
          {'value': 'hu', 'label': 'Hungarian', 'nativeName': 'Magyar'},
          {'value': 'ia', 'label': 'Interlingua', 'nativeName': 'Interlingua'},
          {'value': 'id', 'label': 'Indonesian', 'nativeName': 'Bahasa Indonesia'},
          {'value': 'ie', 'label': 'Interlingue', 'nativeName': 'Originally called Occidental; then Interlingue after WWII'},
          {'value': 'ga', 'label': 'Irish', 'nativeName': 'Gaeilge'},
          {'value': 'ig', 'label': 'Igbo', 'nativeName': 'Asụsụ Igbo'},
          {'value': 'ik', 'label': 'Inupiaq', 'nativeName': 'Iñupiaq, Iñupiatun'},
          {'value': 'io', 'label': 'Ido', 'nativeName': 'Ido'},
          {'value': 'is', 'label': 'Icelandic', 'nativeName': 'Íslenska'},
          {'value': 'it', 'label': 'Italian', 'nativeName': 'Italiano'},
          {'value': 'iu', 'label': 'Inuktitut', 'nativeName': 'ᐃᓄᒃᑎᑐᑦ'},
          {'value': 'ja', 'label': 'Japanese', 'nativeName': '日本語 (にほんご／にっぽんご)'},
          {'value': 'jv', 'label': 'Javanese', 'nativeName': 'basa Jawa'},
          {'value': 'kl', 'label': 'Kalaallisut, Greenlandic', 'nativeName': 'kalaallisut, kalaallit oqaasii'},
          {'value': 'kn', 'label': 'Kannada', 'nativeName': 'ಕನ್ನಡ'},
          {'value': 'kr', 'label': 'Kanuri', 'nativeName': 'Kanuri'},
          {'value': 'ks', 'label': 'Kashmiri', 'nativeName': 'कश्मीरी, كشميري‎'},
          {'value': 'kk', 'label': 'Kazakh', 'nativeName': 'Қазақ тілі'},
          {'value': 'km', 'label': 'Khmer', 'nativeName': 'ភាសាខ្មែរ'},
          {'value': 'ki', 'label': 'Kikuyu, Gikuyu', 'nativeName': 'Gĩkũyũ'},
          {'value': 'rw', 'label': 'Kinyarwanda', 'nativeName': 'Ikinyarwanda'},
          {'value': 'ky', 'label': 'Kirghiz, Kyrgyz', 'nativeName': 'кыргыз тили'},
          {'value': 'kv', 'label': 'Komi', 'nativeName': 'коми кыв'},
          {'value': 'kg', 'label': 'Kongo', 'nativeName': 'KiKongo'},
          {'value': 'ko', 'label': 'Korean', 'nativeName': '한국어 (韓國語), 조선말 (朝鮮語)'},
          {'value': 'ku', 'label': 'Kurdish', 'nativeName': 'Kurdî, كوردی‎'},
          {'value': 'kj', 'label': 'Kwanyama, Kuanyama', 'nativeName': 'Kuanyama'},
          {'value': 'la', 'label': 'Latin', 'nativeName': 'latine, lingua latina'},
          {'value': 'lb', 'label': 'Luxembourgish, Letzeburgesch', 'nativeName': 'Lëtzebuergesch'},
          {'value': 'lg', 'label': 'Luganda', 'nativeName': 'Luganda'},
          {'value': 'li', 'label': 'Limburgish, Limburgan, Limburger', 'nativeName': 'Limburgs'},
          {'value': 'ln', 'label': 'Lingala', 'nativeName': 'Lingála'},
          {'value': 'lo', 'label': 'Lao', 'nativeName': 'ພາສາລາວ'},
          {'value': 'lt', 'label': 'Lithuanian', 'nativeName': 'lietuvių kalba'},
          {'value': 'lu', 'label': 'Luba-Katanga', 'nativeName': ''},
          {'value': 'lv', 'label': 'Latvian', 'nativeName': 'latviešu valoda'},
          {'value': 'gv', 'label': 'Manx', 'nativeName': 'Gaelg, Gailck'},
          {'value': 'mk', 'label': 'Macedonian', 'nativeName': 'македонски јазик'},
          {'value': 'mg', 'label': 'Malagasy', 'nativeName': 'Malagasy fiteny'},
          {'value': 'ms', 'label': 'Malay', 'nativeName': 'bahasa Melayu, بهاس ملايو‎'},
          {'value': 'ml', 'label': 'Malayalam', 'nativeName': 'മലയാളം'},
          {'value': 'mt', 'label': 'Maltese', 'nativeName': 'Malti'},
          {'value': 'mi', 'label': 'Māori', 'nativeName': 'te reo Māori'},
          {'value': 'mr', 'label': 'Marathi (Marāṭhī)', 'nativeName': 'मराठी'},
          {'value': 'mh', 'label': 'Marshallese', 'nativeName': 'Kajin M̧ajeļ'},
          {'value': 'mn', 'label': 'Mongolian', 'nativeName': 'монгол'},
          {'value': 'na', 'label': 'Nauru', 'nativeName': 'Ekakairũ Naoero'},
          {'value': 'nv', 'label': 'Navajo, Navaho', 'nativeName': 'Diné bizaad, Dinékʼehǰí'},
          {'value': 'nb', 'label': 'Norwegian Bokmål', 'nativeName': 'Norsk bokmål'},
          {'value': 'nd', 'label': 'North Ndebele', 'nativeName': 'isiNdebele'},
          {'value': 'ne', 'label': 'Nepali', 'nativeName': 'नेपाली'},
          {'value': 'ng', 'label': 'Ndonga', 'nativeName': 'Owambo'},
          {'value': 'nn', 'label': 'Norwegian Nynorsk', 'nativeName': 'Norsk nynorsk'},
          {'value': 'no', 'label': 'Norwegian', 'nativeName': 'Norsk'},
          {'value': 'ii', 'label': 'Nuosu', 'nativeName': 'ꆈꌠ꒿ Nuosuhxop'},
          {'value': 'nr', 'label': 'South Ndebele', 'nativeName': 'isiNdebele'},
          {'value': 'oc', 'label': 'Occitan', 'nativeName': 'Occitan'},
          {'value': 'oj', 'label': 'Ojibwe, Ojibwa', 'nativeName': 'ᐊᓂᔑᓈᐯᒧᐎᓐ'},
          {
            'value': 'cu',
            'label': 'Old Church Slavonic, Church Slavic, Church Slavonic, Old Bulgarian, Old Slavonic',
            'nativeName': 'ѩзыкъ словѣньскъ'
          },
          {'value': 'om', 'label': 'Oromo', 'nativeName': 'Afaan Oromoo'},
          {'value': 'or', 'label': 'Oriya', 'nativeName': 'ଓଡ଼ିଆ'},
          {'value': 'os', 'label': 'Ossetian, Ossetic', 'nativeName': 'ирон æвзаг'},
          {'value': 'pa', 'label': 'Panjabi, Punjabi', 'nativeName': 'ਪੰਜਾਬੀ, پنجابی‎'},
          {'value': 'pi', 'label': 'Pāli', 'nativeName': 'पाऴि'},
          {'value': 'fa', 'label': 'Persian', 'nativeName': 'فارسی'},
          {'value': 'pl', 'label': 'Polish', 'nativeName': 'polski'},
          {'value': 'ps', 'label': 'Pashto, Pushto', 'nativeName': 'پښتو'},
          {'value': 'pt', 'label': 'Portuguese', 'nativeName': 'Português'},
          {'value': 'qu', 'label': 'Quechua', 'nativeName': 'Runa Simi, Kichwa'},
          {'value': 'rm', 'label': 'Romansh', 'nativeName': 'rumantsch grischun'},
          {'value': 'rn', 'label': 'Kirundi', 'nativeName': 'kiRundi'},
          {'value': 'ro', 'label': 'Romanian, Moldavian, Moldovan', 'nativeName': 'română'},
          {'value': 'ru', 'label': 'Russian', 'nativeName': 'русский язык'},
          {'value': 'sa', 'label': 'Sanskrit (Saṁskṛta)', 'nativeName': 'संस्कृतम्'},
          {'value': 'sc', 'label': 'Sardinian', 'nativeName': 'sardu'},
          {'value': 'sd', 'label': 'Sindhi', 'nativeName': 'सिन्धी, سنڌي، سندھی‎'},
          {'value': 'se', 'label': 'Northern Sami', 'nativeName': 'Davvisámegiella'},
          {'value': 'sm', 'label': 'Samoan', 'nativeName': 'gagana faa Samoa'},
          {'value': 'sg', 'label': 'Sango', 'nativeName': 'yângâ tî sängö'},
          {'value': 'sr', 'label': 'Serbian', 'nativeName': 'српски језик'},
          {'value': 'gd', 'label': 'Scottish Gaelic; Gaelic', 'nativeName': 'Gàidhlig'},
          {'value': 'sn', 'label': 'Shona', 'nativeName': 'chiShona'},
          {'value': 'si', 'label': 'Sinhala, Sinhalese', 'nativeName': 'සිංහල'},
          {'value': 'sk', 'label': 'Slovak', 'nativeName': 'slovenčina'},
          {'value': 'sl', 'label': 'Slovene', 'nativeName': 'slovenščina'},
          {'value': 'so', 'label': 'Somali', 'nativeName': 'Soomaaliga, af Soomaali'},
          {'value': 'st', 'label': 'Southern Sotho', 'nativeName': 'Sesotho'},
          {'value': 'es', 'label': 'Spanish; Castilian', 'nativeName': 'español, castellano'},
          {'value': 'su', 'label': 'Sundanese', 'nativeName': 'Basa Sunda'},
          {'value': 'sw', 'label': 'Swahili', 'nativeName': 'Kiswahili'},
          {'value': 'ss', 'label': 'Swati', 'nativeName': 'SiSwati'},
          {'value': 'sv', 'label': 'Swedish', 'nativeName': 'svenska'},
          {'value': 'ta', 'label': 'Tamil', 'nativeName': 'தமிழ்'},
          {'value': 'te', 'label': 'Telugu', 'nativeName': 'తెలుగు'},
          {'value': 'tg', 'label': 'Tajik', 'nativeName': 'тоҷикӣ, toğikī, تاجیکی‎'},
          {'value': 'th', 'label': 'Thai', 'nativeName': 'ไทย'},
          {'value': 'ti', 'label': 'Tigrinya', 'nativeName': 'ትግርኛ'},
          {'value': 'bo', 'label': 'Tibetan Standard, Tibetan, Central', 'nativeName': 'བོད་ཡིག'},
          {'value': 'tk', 'label': 'Turkmen', 'nativeName': 'Türkmen, Түркмен'},
          {'value': 'tl', 'label': 'Tagalog', 'nativeName': 'Wikang Tagalog, ᜏᜒᜃᜅ᜔ ᜆᜄᜎᜓᜄ᜔'},
          {'value': 'tn', 'label': 'Tswana', 'nativeName': 'Setswana'},
          {'value': 'to', 'label': 'Tonga (Tonga Islands)', 'nativeName': 'faka Tonga'},
          {'value': 'tr', 'label': 'Turkish', 'nativeName': 'Türkçe'},
          {'value': 'ts', 'label': 'Tsonga', 'nativeName': 'Xitsonga'},
          {'value': 'tt', 'label': 'Tatar', 'nativeName': 'татарча, tatarça, تاتارچا‎'},
          {'value': 'tw', 'label': 'Twi', 'nativeName': 'Twi'},
          {'value': 'ty', 'label': 'Tahitian', 'nativeName': 'Reo Tahiti'},
          {'value': 'ug', 'label': 'Uighur, Uyghur', 'nativeName': 'Uyƣurqə, ئۇيغۇرچە‎'},
          {'value': 'uk', 'label': 'Ukrainian', 'nativeName': 'українська'},
          {'value': 'ur', 'label': 'Urdu', 'nativeName': 'اردو'},
          {'value': 'uz', 'label': 'Uzbek', 'nativeName': 'zbek, Ўзбек, أۇزبېك‎'},
          {'value': 've', 'label': 'Venda', 'nativeName': 'Tshivenḓa'},
          {'value': 'vi', 'label': 'Vietnamese', 'nativeName': 'Tiếng Việt'},
          {'value': 'vo', 'label': 'Volapük', 'nativeName': 'Volapük'},
          {'value': 'wa', 'label': 'Walloon', 'nativeName': 'Walon'},
          {'value': 'cy', 'label': 'Welsh', 'nativeName': 'Cymraeg'},
          {'value': 'wo', 'label': 'Wolof', 'nativeName': 'Wollof'},
          {'value': 'fy', 'label': 'Western Frisian', 'nativeName': 'Frysk'},
          {'value': 'xh', 'label': 'Xhosa', 'nativeName': 'isiXhosa'},
          {'value': 'yi', 'label': 'Yiddish', 'nativeName': 'ייִדיש'},
          {'value': 'yo', 'label': 'Yoruba', 'nativeName': 'Yorùbá'},
          {'value': 'za', 'label': 'Zhuang, Chuang', 'nativeName': 'Saɯ cueŋƅ, Saw cuengh'}
        ],
        chosenLanguage: ''
      }
    },
    methods: {
      openDialog () {
        this.dialog = true
      },
      close () {
        this.dialog = false
        this.getHomographDialog = false
      },
      openHomograph () {
        this.getHomographDialog = true
      },
      getUser () {
        let _this = this
        this.$http.post('https://byui-homograph.appspot.com/newUser', {
          language: _this.chosenLanguage.value,
          location: 'US'
        }).then(response => {
          _this.userId = response.body.uuid
          _this.token = response.body.request_key
          _this.getHomograph()
        })
      },
      getHomograph () {
        let _this = this
        this.getHomographDialog = false
        this.clear()
        this.$http.post('https://byui-homograph.appspot.com/getHomographGame', {
          difficulty_low: _this.difficulty,
          difficulty_high: _this.difficulty,
          frequency: _this.frequency,
          uuid: _this.userId,
          token: _this.token
        }).then(response => {
          let receivedData = response.body
          _this.homograph = receivedData.altered_string
          _this.originalText = receivedData.string
          _this.homographData = receivedData.homographs
          _this.token = receivedData.token
          _this.printHomographDataToConsole()
        })
      },
      homographClick (index, e) {
        if (this.selected.includes(index)) {
          let selectedIndex = this.selected.indexOf(index)
          this.selected.splice(selectedIndex, 1)
          e.target.parentElement.style.backgroundColor = null
          let matchIndex = _.findIndex(this.matches, function (o) { return o.word_index === index })
          if (matchIndex !== -1) {
            this.matches.splice(matchIndex, 1)
          }
        } else {
          e.target.parentElement.style.backgroundColor = '#FFEBEE'
          this.selected.push(index)
          this.checkMatch(index)
          this.selected = _.sortBy(this.selected, function (o) { return o })
        }
      },
      checkMatch (index) {
        let found = this.findMatch(index)
        if (found !== -1) {
          this.matches.push({
            word: this.homograph[index],
            index: this.homographData[found].character_index,
            word_index: index
          })
        }
        this.matches = _.sortBy(this.matches, function (o) { return o.word_index })
      },
      findMatch (index) {
        return _.findIndex(this.homographData, function (o) {
          return o.word_index === index
        })
      },
      printHomographDataToConsole () {
        console.info(this.homographData)
        _.each(this.homographData, function (o) {
          console.info('Look For ' + o.character_changed_to + ' in word "' + o.original_word + '" at index ' + String(o.word_index))
        })
      },
      clear () {
        this.homograph = []
        this.originalText = []
        this.homographData = []
        this.selected = []
        this.matches = []
      }
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss">
  h1, h2 {
    font-weight: normal;
  }

  ul {
    list-style-type: none;
    padding: 0;
  }

  li {
    display: inline-block;
    margin: 0 10px;
  }

  a {
    color: #42b983;
  }

  button.homograph.mu-flat-button {
    min-width: 0;
    .mu-flat-button-wrapper {
      .mu-flat-button-label {
        font-family: 'arial-unicode', sans-serif;
        text-decoration: none;
        text-transform: none;
        padding: 0 5px;
      }
    }
  }

  .sequence {
    margin-top: 15px;
  }

  .mu-dialog {
    .mu-dialog-body {
      height: 350px;
    }
  }

  .controls {
    .mu-raised-button {
      margin: 0 5px;
    }
    margin-bottom: 10px;
  }
</style>
