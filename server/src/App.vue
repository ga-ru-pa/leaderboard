<template>
  <div id="app">
    <div class="has-text-centered head">
        <p class="is-size-3">ガルパ非公式スコアランキング</p>
    </div>
    <label class="label garupa">タイトルを選択</label>
    <div class="control">
      <select class="select garupa" v-model="selected_title_id">
        <option v-for="t in titles" :key="t.id" :value="t.id">{{t.id + ": " + t.title}}</option>
      </select>
    </div>

    <hr/>
    <div v-if="selected_title_id === null">
      タイトルを選択すると、ランキングが表示されます。
    </div>
    <div v-else>
      <label class="label garupa">参加方法</label>
      <div class="control">
        <ul>
          <li>①リザルトを添付し、 ②本文に総合力とスキル内訳(（例）125-120-100-100-100 順番は問わない) を含めて@garupa_yatteru に連絡してください。</li>
          <li>こちらで確認した内容を反映します。手作業のため、漏れやミスがあるかもしれません。お気付きの場合はその旨ご連絡ください。</li>
        </ul>
      </div>

      <label class="label garupa">ルール</label>
      <div class="control">
        <ul>
          <li>スコアランキングと、スコア率ランキングの2種類があります。</li>
          <li>スコア率は、達成スコア / (提出された編成でリズム通りにAPした場合のスコアの最大値) で算出されます。編成によって差が生じるので、参考程度にお考えください。</li>
        </ul>
      </div>
      <div class="rankings">
        <label class="label garupa">スコアランキング</label>
        <div class="control" v-for="(row, index) in ranking1" :key="index">
          <div style="display:inline-block;width:600px;">
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
        <label class="label garupa">スコア率ランキング</label>
        <div class="control" v-for="(row, index) in ranking2" :key="row.name">
          <div style="display:inline-block;width:600px;">
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
      titles: null,
      scores: null,
      ranking1: null,
      ranking2: null
    }
  },
  async mounted () {
    this.titles = (await this.axios.get('https://raw.githubusercontent.com/ga-ru-pa/leaderboard-data/main/titles.json')).data.titles
    this.selected_title_id = this.titles[0].id
  },
  watch: {
    async selected_title_id () {
      this.scores = (await this.axios.get('https://raw.githubusercontent.com/ga-ru-pa/leaderboard-data/main/' + this.selected_title_id + '/scores.json')).data.scores
      this.ranking1 = this.scores.map(list => ({...list})).sort((a, b) => b.score !== a.score ? b.score - a.score : a.unixtime - b.unixtime)
      this.ranking2 = this.scores.sort((a, b) => b.score * a.jap_score - a.score * b.jap_score)
    }
  }
}
</script>

<style>
body {
  margin: 5px;
  width: 600px;
}

label.garupa {
  color: white;
  background: #fc3f73;
  border-radius: 15px;
  padding: 0 15px;
  margin: 10px 0;
  display: inline-table;
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
  border-radius: 5px;
  border: solid 1px;
  margin: 5px;
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
  font-size: 12px;
}
div.ranking-name-rating {
  display: flex;
  border-bottom: dotted;
  border-color: #CCC;
  align-items: center;
  justify-content: space-between;
}

div.ranking-name {
  font-size: 18px;
}

div.ranking-score {
  font-size: 15px;
  color:#fff;
  background:#777777;
  border-radius:15px;
  padding: 0 15px;
  margin: 5px;
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
  font-size: 18px;
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
</style>
