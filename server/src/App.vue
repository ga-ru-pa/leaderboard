<template>
  <div id="app">
    <div class="has-text-centered head">
      <p class="titleheader">ガルパ非公式 スコアランキング</p>
    </div>
    <label class="label garupa">タイトルを選択</label>
    <div class="control">
      <select class="select garupa" v-model="selected_title_id">
        <option v-for="t in titles" :key="t.id" :value="t.id">{{t.id + ": " + t.title}}</option>
      </select>
    </div>

    <hr/>
    <div v-if="selected_title_id === null">
      Loading
    </div>
    <div v-else>
      <label class="label garupa">開催期間</label>
      <div class="control">
        {{start_datetime.toLocaleString()}} ~ {{end_datetime.toLocaleString()}}
      </div>
      <div class="control">
        <label class="label garupa">参加方法・ルール</label>
        <details>
          <summary>こちら</summary>
          <div class="control">
            <label class="label garupa">参加方法</label>
            <summary>以下の内容を@garupa_yatteru に送ってください。</summary>
            <table class="table is-striped participation">
              <tr>
                <th>項目</th>
                <th>必須</th>
                <th>備考</th>
                <th>例</th>
              </tr>
              <tr>
                <td>スコア</td>
                <td>*</td>
                <td>リザルトのスクリーンショットなど。</td>
                <td>2012345</td>
              </tr>
              <tr>
                <td>名前</td>
                <td></td>
                <td>好きな文字を利用可。なければこちらで適当につけます。</td>
                <td>👮</td>
              </tr>
              <tr>
                <td>編成</td>
                <td></td>
                <td>参考情報としてランキングに表示されます。</td>
                <td>クールパスパレ</td>
              </tr>
              <tr>
                <td>総合力</td>
                <td>*</td>
                <td>スコア率の計算に使います。</td>
                <td>297691</td>
              </tr>
              <tr>
                <td>スキル</td>
                <td>*</td>
                <td>スコア率の計算に使います。</td>
                <td>125-120-120-115-115</td>
              </tr>
            </table>
            ※手作業で更新するため、漏れやミスがあるかもしれません。お気付きの場合はご連絡ください。
          </div>

          <label class="label garupa">ルール</label>
          <div class="control">
            <ul>
              <li>
                ①<a href="#ranking1">スコア率ランキング</a>と、
                ②<a href="#ranking2">スコアランキング</a>の2種類があります。</li>
              <li>①スコア率ランキング: 編成を問わず遊びたい方向け。スコア率は、(達成スコア) / (提出された編成でリズム通りにAPした場合のスコアの最大値) で算出されます。編成によって上限値が異なります。</li>
              <li>②スコアランキング: 単純にスコアで比較します。</li>
            </ul>
          </div>
        </details>
      </div>

      <div class="rankings">
        <label id="ranking1" class="label garupa">スコア率ランキング</label>
        <div class="control" v-for="(row, index) in ranking2" :key="row.name">
          <div class="ranking-container">
            <div class="ranking-rank ribbon">
              <h3>{{ index + 1 }}位</h3>
            </div>
            <div style="width:100%">
              <div class="ranking-name-rating">
                <div class="ranking-name">{{ row.name }}</div>
              </div>
              <div class="ranking-team-score">
                <div class="ranking-team">{{ row.team + " / 総合力: " + row.power + " / スキル: " + row.skill }}</div>
                <div class="ranking-score">{{ (row.score*100/row.jap_score).toFixed(3) + "%" }}</div>
              </div>
            </div>
          </div>
        </div>
        <label id="ranking2" class="label garupa">スコアランキング</label>
        <div class="control" v-for="(row, index) in ranking1" :key="index">
          <div class="ranking-container">
            <div class="ranking-rank ribbon">
              <h3>{{ index + 1 }}位</h3>
            </div>
            <div style="width:100%">
              <div class="ranking-name-rating">
                <div class="ranking-name">{{ row.name }}</div>
              </div>
              <div class="ranking-team-score">
                <div class="ranking-team">{{ row.team + " / 総合力: " + row.power + " / スキル: " + row.skill }}</div>
                <div class="ranking-score">{{ row.score }}</div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data () {
    return {
      selected_title_id: null,
      start_datetime: null,
      end_datetime: null,
      titles: null,
      titles_rev_order: null,
      scores: null,
      ranking1: null,
      ranking2: null
    }
  },
  async mounted () {
    this.titles = (await this.axios.get('https://raw.githubusercontent.com/ga-ru-pa/leaderboard-data/main/titles.json')).data.titles
    this.selected_title_id = this.titles[0].id
    this.titles_rev_order = this.titles.map(list => ({...list})).sort((a, b) => a.id - b.id)
  },
  watch: {
    async selected_title_id () {
      this.scores = (await this.axios.get('https://raw.githubusercontent.com/ga-ru-pa/leaderboard-data/main/' + this.selected_title_id + '/scores.json')).data.scores
      this.ranking1 = this.scores.map(list => ({...list})).sort((a, b) => b.score !== a.score ? b.score - a.score : a.unixtime - b.unixtime)
      this.ranking2 = this.scores.sort((a, b) => b.score * a.jap_score !== a.score * b.jap_score ? b.score * a.jap_score - a.score * b.jap_score : a.unixtime - b.unixtime)
      this.start_datetime = new Date(this.titles_rev_order[this.selected_title_id].start_datetime.replace(/-/g, '/'))
      this.end_datetime = new Date(this.titles_rev_order[this.selected_title_id].end_datetime.replace(/-/g, '/'))
    }
  }
}
</script>

<style>
html {
  font-size: calc(0.75rem + ((1vw - 3.2px) * 0.125));
  min-height: 0vw;
}
body {
  padding: 5px;
  width: 100%;
}

label.garupa {
  color: white;
  background: #fc3f73;
  border-radius: 15px;
  padding: 0 15px;
  margin: 10px 0;
  display: inline-table;
  font-size: 1rem;
}

select.garupa {
  background: #dcdcdc;
  width: 100%;
  border-radius: 4px;
  padding: 0 5px;
}

select {
  font-size: 1rem;
  box-shadow: inset 1px 1px #333;
}

input[type=text].garupa {
  background: #dcdcdc;
  box-shadow: inset 1px 1px #333;
}

textarea.garupa {
  background: #dcdcdc;
  box-shadow: inset 1px 1px #333;
}

hr {
  background: #fc3f73;
  margin-top: 10px;
  margin-bottom: 5px;
}

.horizontal {
  display: flex;
}

.for-checkbox {
  width: fit-content;
  padding: 0 5px;
}

.button.garupa {
  background: #fc3f73;
  margin: 5px;
}

div.ranking-container {
  display: inline-block;
  width: 100%;
  max-width: 600px;
  border-radius: 5px;
  border: solid 1px;
  margin: 5px 0;
  padding: 5px;
  display: flex;
  align-items: center;
}
div.ranking-rank {
  width: 80px;
  margin: 0 5px;
}
div.ranking-team-score {
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-size: 0.7rem;
}
div.ranking-name-rating {
  display: flex;
  border-bottom: dotted;
  border-color: #CCC;
  align-items: center;
  justify-content: space-between;
}

div.ranking-name {
  font-size: 1rem;
  font-weight: bold;
}

div.ranking-score {
  font-size: 1rem;
  color:#fff;
  background:#777777;
  border-radius:15px;
  padding: 0 15px;
  margin: 1px 0;
  width: 100px;
  vertical-align: center;
}

.ribbon {
  display: inline-block;
  position: relative;
  height: 20px;
  line-height: 20px;
  text-align: center;
  padding: 0 20px;
  font-size: 1rem;
  background: #ff6fa3;
  color: #FFF;
  box-sizing: border-box;
}

.ribbon:before, .ribbon:after {
  position: absolute;
  content: '';
  width: 0px;
  height: 0px;
  z-index: 1;
}

.ribbon:before {
  top: 0;
  left: 0;
  border-width: 10px 0px 10px 8px;
  border-color: transparent transparent transparent #fff;
  border-style: solid;
}

.ribbon:after {
  top: 0;
  right: 0;
  border-width: 10px 8px 10px 0px;
  border-color: transparent #fff transparent transparent;
  border-style: solid;
}

p.titleheader {
  word-break: keep-all;
  font-size: 1.5rem;
  font-weight: bold;
}

table.participation {
  font-size: 0.6rem;
}
</style>
