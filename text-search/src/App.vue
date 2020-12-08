<template>
	<div id="app">
		<b-container>
			<h2>Search text</h2>
			<h5>With having fat-fingers like ćevapčići </h5>

			
			<b-input-group prepend="Phrase:" class="mt-3">
				<b-form-input v-model="phrase"></b-form-input>
				<b-input-group-append>
					<b-button variant="success" @click="addToPhraseList(phrase)">Search</b-button>
				</b-input-group-append>
			</b-input-group>
<!-- 			<ul id="example-1" class="mt-3">
				<li v-for="item in phraseList" :key="item.phrase">
					{{ item.phrase }}
				</li>
			</ul>
 -->
			
				

			<p ref="para" class="mt-4">
								Far out in the uncharted backwaters of the unfashionable end of 
				the western spiral arm of the Galaxy lies a small unregarded yellow 
				sun. 
				Orbiting this at a distance of roughly ninety`two million miles is an 
				utterly insignificant little blue green planet whose ape`descended life 
				forms are so amazingly primitive that they still think digital watches 
				are a pretty neat idea. 
				This planet has ʹ or rather had ʹ a problem, which was this: most 
				of the people on it were unhappy for pretty much of the time. Many 
				solutions were suggested for this problem, but most of these were 
				largely concerned with the movements of small green pieces of paper, 
				which is odd because on the whole it wasn't the small green pieces of 
				paper that were unhappy. 
			</p>

			
			<b-table class="mt-5" hover :items="items" :fields="fields" style="color: lightgrey"></b-table>
		</b-container>
	</div>
</template>

<script>

export default {
	name: 'App',
		data() {
			return {
				fields: ['phrase', 'in_text?'],
				items: [],
				text: 'insignificant little blue green planet',
				phrase: '',
				phraseList: [],
			}
		},
	methods: {
		addToPhraseList(text) {
			var includes = false
			for (let i = 0; i < this.phraseList.length; i++) {
				if (this.phraseList[i].phrase == text) {
					includes = true
				}
			}

			if (text == '' || includes) {
				alert("Input is empty or already included entered!")
			} else {	
				this.phraseList.push({ "phrase": text })
				var found = this.search(text)
				if(found) {
					this.items.push({ "phrase": text, "in_text?": "Yes"})
				} else {
					this.items.push({ "phrase": text, "in_text?": "No"})
				}
			}


		},
		search(text) {
			var p = this.$refs.para.innerHTML.split(" ")
			var t = text.split(" ")

			var found = false
			for (let i = 0;  i < p.length; i++) {
				var dist = 0
				dist = this.levensthein(p.slice(i, i + t.length), t)
				//ne znam koji bi bio optimalan score
				if (dist < t.length) {
					found = true
				}
			}
			return found
		},
		levensthein(l1, l2) {

			var s1 = l1.join(" ")
			var s2 = l2.join(" ")
			
			//inicijalizuj matricu sa jednim vise redom i kolonom (za prefix koji oznacava prazan string)
			//i ispuni sa nulama
			let matrix = [];
			for (let i = 0; i < s1.length + 1; i++) {
				matrix[i] = new Array(s2.length + 1).fill(0);
			}

			//popuni prvu kolonu sa vrijednostima od 1 do duzina-prvog-stringa
			for (let i = 1; i < s1.length + 1; i++) {
				matrix[i][0] = i
			}

			
			//popuni drugu kolonu sa vrijednostima od 1 do duzina-drugog-stringa
			for (let j = 1; j < s2.length + 1; j++) {
				matrix[0][j] = j
			}

			for (let j = 1; j < s2.length + 1; j++) {
				for (let i = 1;  i < s1.length + 1; i++) {
					var subCost = 0
					//ako se slova razlikuju, cijena zamjene ce biti 1
					if (s1[i] != s2[j]) {
						subCost = 1
					}
					//glavna formula
					matrix[i][j] = Math.min(matrix[i-1][j] + 1, matrix[i][j-1] + 1, matrix[i-1][j-1] + subCost)
				}
			}
			//vrati poslednju vrijednost u dijagonali matrice
			//koja oznacava koliko operacija (add, remove, update) 
			//bi trebalo da se izvrsi nad prvim stringom da bi bio jednak drugom
			return matrix[s1.length][s2.length]
		}
	}
}
</script>

<style>
	#app {
		font-family: Avenir, Helvetica, Arial, sans-serif;
		-webkit-font-smoothing: antialiased;
		-moz-osx-font-smoothing: grayscale;
		text-align: center;
		color: #cacaca;
		margin-top: 60px;
	}
</style>
