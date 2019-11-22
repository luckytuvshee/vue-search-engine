<template>
  <div id="app">
    <table id="t">
      <thead>
        <tr>
          <th>term</th>
          <th v-for="(num, index) in docNumber" :key="index">{{ getDocTitle(num) }}</th>
          <th>df</th>
          <th>idf</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(d, index) in dic" :key="index">
          <td>{{ word[index] }}</td>

          <td
            v-for="(count, index) in d"
            :key="index"
          >{{ convertFixed(wordWeight(count) * getIdf(getDf(d))) }}</td>

          <td>{{ getDf(d) }}</td>
          <td>{{ getIdf(getDf(d)) }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
export default {
  name: "app",
  mounted() {

    var cos = [];

    var tableElem = window.document.getElementById("t");
    var tableBody = tableElem.getElementsByTagName("tbody").item(0);

    var tRows = tableBody.rows.length;

    for (let d = 1; d <= 10; d++) {
      var sum1 = 0;
      var sum2 = 0;
      for (let i = 1; i < tRows - 1; i++) {
        var thisTrElem = tableBody.rows[i];
        var thisTdElem = thisTrElem.cells[d];
        var query = thisTrElem.cells[11];

        var thisTextNode = thisTdElem.childNodes.item(0);
        var queryTextNode = query.childNodes.item(0);
        
        sum1 += thisTextNode.nodeValue * queryTextNode.nodeValue;
        sum2 += Math.sqrt(thisTextNode.nodeValue * thisTextNode.nodeValue) * Math.sqrt(queryTextNode.nodeValue * queryTextNode.nodeValue)
      }
    

      cos.push(sum2 == 0 ? 0 : parseFloat(parseFloat((sum1 + "" / sum2 + "")).toFixed(5)) );
    }
    console.log("Косинус төсөө")
    console.log(cos.sort((a, b) => b - a));
  },

  methods: {
    getDocTitle(num) {
      return num === this.docs.length ? "query" : "doc " + num;
    },

    getDf(arr) {
      let count = 0;
      arr.forEach(x => {
        if (x !== 0) count++;
      });
      return count;
    },

    getIdf(df) {
      // this.docs.length - 1; minus -1 is query, so it shouldn't be counted
      return this.convertFixed(Math.log10(this.docs.length - 1 / df));
    },

    wordWeight(word) {
      return word <= 0 ? 0 : this.convertFixed(Math.log10(word) + 1);
    },

    convertFixed(num) {
      return num === 0 ? 0 : num.toFixed(3);
    }
  },
  computed: {
    word: function() {
      return this.wordsText.split(" ");
    },

    docNumber: function() {
      return this.docs.length;
    },

    dic: function() {
      var countArr = [];
      this.wordsText.split(" ").forEach(word => {
        var countNestedArr = [];

        this.docs.forEach(doc => {
          var count = 0;
          var eachDoc = doc.split(" ");
          for (let i = 0; i < eachDoc.length; i++) {
            if (eachDoc[i].trim() == word.trim()) {
              count++;
            }
          }
          countNestedArr.push(count);
        });
        countArr.push(countNestedArr);
      });
      return countArr;
    }
  },
  data() {
    return {
      docCoordinates: [],
      wordsText: `аав анги ах ахлах бага багш барих бас баярлах баяртай би бие бол болох бороо гамнах гараа гэж гэсэн даанч дараа дэвших дөө заавал зугтах зэвсэг миний минь мөн наагуур найз намираа нар наслах ногоон ном нэрлэх нямбай нүүр одох сайхан солонго сонсох сурвал сургууль таатай тамир татах тоглох тустай тэнхээ тэр төгсөх угаах удаан улаан урт уулзах хайрлах харандаа харин хийх хурга хурдлах хэлэх хүн хүндэтгэх хүр хүү хөх хөөрхөн цааш цагираг цэвэрхэн ч чийрэг шамдах шар шиг эгч эрдэм эрүүл ээж ээх явах үг өглөө өнгө өнгөрөх өөрт`,
      docs: [
        `намираа дараа нар таатай ээх өнгөрөх бороо наагуур өнгө солонго татах`,
        `тэр солонго барих гэж тэнхээ би хурдлах цагираг солонго харин цааш зугтах одох`,
        `ахлах анги төгсөх бага сургууль дэвших ах эгч нар минь баяртай эрдэм ном шамдах`,
        `тамир хийх бие болох эрүүл чийрэг явах ээж аав баярлах`,
        `урт удаан наслах гэсэн хүн үг сонсох явах өөрт заавал тустай эрдэм ном сурвал`,
        `багш баяртай бага минь найз дараа өглөө уулзах даанч сайхан тоглох`,
        `миний шар харандаа бол зэвсэг би хайрлах гамнах`,
        `хүр бороо хурга мөн ч хөөрхөн дөө нүүр гараа угаах хүү бас`,
        `угаах хурга хүндэтгэх хөөрхөн гэж хэлэх шиг цэвэрхэн хүү нямбай нэрлэх`,
        `өнгө улаан харандаа хөх миний ногоон`,
        `намираа бороо тоглох сайхан гэж хэлэх хүү бас хөөрөн дөө`
      ]
    };
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

th,
td {
  border: 1px solid black;
  padding: 2px 5px;
}
</style>
