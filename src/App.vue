<template>
  <div id="app">
    <div class="searchEngine">
      <h2>Шүлгийн хайлтын систем</h2>
      <div class="search">
        <input v-model="query" class="input" type="text" />
        <button @click="calcCos">Хайх</button>
      </div>

      <div class="result">
        <div>
          <button @click="ranking">Rank тогтоох</button>

          <div v-if="rankingResult">
            <p>Хамгийн төстэй 3 баримт</p>
            <ol>
              <li v-for="c in nr3" :key="c.doc">doc: {{c.doc}}, cos: {{ c.cos }}</li>
            </ol>
          </div>
        </div>
        <div v-if="!rankingResult && showResult">
          <p>{{ error }}</p>
          <p>Хайлтын илэрц</p>
          <ol>
            <li v-for="c in cos" :key="c.doc">
              doc: {{c.doc}}, cos: {{ c.cos }}
              <br />
              {{ docs[c.doc - 1] }}
            </li>
          </ol>

          <div>
            <h4>Нийт баримт: {{ docs.length }}</h4>
            <h4>Хайлтанд гарч ирсэн баримт: {{ cos.length }}</h4>
            <h4>Хамаатай баримтууд: {{ relevantDocs }}</h4>

            <table>
              <tr>
                <td></td>
                <td>Хамаатай</td>
                <td>Хамаагүй</td>
                <td></td>
              </tr>
              <tr>
                <td>Гарсан</td>
                <td>{{ tp }}</td>
                <td>{{ fp }}</td>
                <td>{{ cos.length }}</td>
              </tr>
              <tr>
                <td>Үлдсэн</td>
                <td>{{ fn }}</td>
                <td>{{ tn }}</td>
                <td>{{ docs.length - cos.length }}</td>
              </tr>
              <tr>
                <td></td>
                <td>{{ tp + fn }}</td>
                <td>{{ fp + tn }}</td>
                <td>{{ docs.length }}</td>
              </tr>
            </table>
            <hr />

            <h4>Ололт: {{ tp / (tp + fp) }} %</h4>
            <h4>Амжилт: {{ tp / (tp + fn) }} %</h4>
            <h4>Нарийвчлал: {{ (tp + tn) / (tp + fp + fn + tn) }} %</h4>
          </div>
        </div>
      </div>
    </div>

    <table v-if="showResult" id="t">
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
  methods: {
    ranking: function() {
      this.showResult = true;
      this.rankingResult = true;

      setTimeout(() => {
        var tableElem = window.document.getElementById("t");
        var tableBody = tableElem.getElementsByTagName("tbody").item(0);

        var tRows = tableBody.rows.length;

        var maxCos = [];

        for (let d = 1; d <= this.docs.length - 1; d++) {
          var cos = [];
          var sum1 = 0;
          var sum2 = 0;
          var sum3 = 0;

          for (let y = 1; y <= this.docs.length - 1; y++) {
            if (d == y) continue;

            for (let i = 0; i < tRows; i++) {
              var thisTrElem = tableBody.rows[i]; // term
              var thisTdElem = thisTrElem.cells[d]; // term coordinate
              var thisTdElem2 = thisTrElem.cells[y];

              var thisTextNode = thisTdElem.childNodes.item(0);
              var this2TextNode = thisTdElem2.childNodes.item(0);

              sum1 += thisTextNode.nodeValue * this2TextNode.nodeValue;
              sum2 += Math.pow(thisTextNode.nodeValue, 2);
              sum3 += Math.pow(this2TextNode.nodeValue, 2);
            }

            sum2 = Math.sqrt(sum2) * Math.sqrt(sum3);

            if (sum1 / sum2 !== 0)
              cos.push({ doc: d, cos: (sum1 / sum2).toFixed(3) });
          }

          maxCos.push(cos.sort((a, b) => b.cos - a.cos)[0]);
        }

        function compare(a, b) {
          if (a.cos < b.cos) {
            return 1;
          }
          if (a.cos > b.cos) {
            return -1;
          }
          return 0;
        }

        this.nr3 = maxCos.sort(compare).slice(0, 3);
      }, 100);
    },
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
    },

    calcCos() {
      this.rankingResult = false;
      if (this.query.trim() == "") {
        this.error = "Утга оруулна уу";
        return;
      }

      this.showResult = true;
      this.error = "";
      this.docs.push(this.query);

      setTimeout(() => {
        var cos = [];

        var tableElem = window.document.getElementById("t");
        var tableBody = tableElem.getElementsByTagName("tbody").item(0);

        var tRows = tableBody.rows.length;

        for (let d = 1; d <= this.docs.length - 1; d++) {
          var sum1 = 0;
          var sum2 = 0;
          var sum3 = 0;
          for (let i = 0; i < tRows; i++) {
            var thisTrElem = tableBody.rows[i]; // term
            var thisTdElem = thisTrElem.cells[d]; // term coordinate
            var query = thisTrElem.cells[this.docs.length];

            var thisTextNode = thisTdElem.childNodes.item(0);
            var queryTextNode = query.childNodes.item(0);

            sum1 += thisTextNode.nodeValue * queryTextNode.nodeValue;
            sum2 += Math.pow(thisTextNode.nodeValue, 2);
            sum3 += Math.pow(queryTextNode.nodeValue, 2);
          }

          sum2 = Math.sqrt(sum2) * Math.sqrt(sum3);

          if (sum1 / sum2 !== 0)
            cos.push({ doc: d, cos: (sum1 / sum2).toFixed(3) });
        }

        function compare(a, b) {
          if (a.cos < b.cos) {
            return 1;
          }
          if (a.cos > b.cos) {
            return -1;
          }
          return 0;
        }

        this.cos = cos.sort(compare);
        this.docs.pop();

        var relevantCount = 0;
        this.cos.forEach(docs => {
          if (this.relevantDocs.includes(docs.doc)) {
            relevantCount++;
          }
        });

        var cosValue = this.cos.map(cos => cos.doc);

        var negative = Array.from(Array(10), (_, x) => x + 1).filter(
          num => !cosValue.includes(num)
        );

        var nonRelevantCount = 0;
        negative.forEach(doc => {
          if (this.relevantDocs.includes(doc)) {
            nonRelevantCount++;
          }
        });

        console.log(negative);

        this.tp = relevantCount;
        this.fp = this.cos.length - this.tp;
        this.fn = nonRelevantCount;
        this.tn = negative.length - nonRelevantCount;
      }, 100);
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
      cos: [],
      nr3: [],
      relevantDocs: [1, 8, 10],
      tp: 0,
      fp: 0,
      fn: 0,
      tn: 0,
      error: "",
      query: "намираа дараа нар таатай ээх өнгөрөх",
      rankingResult: false,
      showResult: false,
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
        `өнгө улаан харандаа хөх миний ногоон`
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
  font-family: monaco;
}

.searchEngine {
  border: 1px solid #333;
  border-radius: 5px;
  display: grid;
  max-width: 750px;
  margin: auto;
  margin-bottom: 50px;
  padding: 20px;
}

.search {
  display: grid;
  grid-template-columns: 90% 10%;
  margin-bottom: 10px;
}

button:hover {
  cursor: pointer;
}

.input {
  height: 30px;
  border: 1px solid #333;
  border-radius: 2px;
}

.result {
  text-align: left;
}

#t {
  margin: auto;
}

th,
td {
  border: 1px solid black;
  padding: 2px 5px;
}
</style>
