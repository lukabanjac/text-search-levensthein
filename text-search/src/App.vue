<template>
	<div id="app">
		<b-container>
			<h2 style="text-weight: bold; color: pink; font-weight: bold;">Search text</h2>
			<h5>With having fat-fingers like ćevapčići </h5>

			
			<b-input-group prepend="Phrase:" class="mt-3">
				<b-form-input style="background: lightgray;" v-model="phrase"></b-form-input>
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
			
				

			<p ref="para" v-html="styled_paragraph_text" class="mt-4">
				{{styled_paragraph_text}}
			</p>

			
			<b-table small striped class="mt-5" hover :items="items" :fields="fields" style="color: lightgrey"></b-table>
		</b-container>
	</div>
</template>

<script>
import txt from 'raw-loader!./assets/text.txt';

export default {
	name: 'App',
		data() {
			return {
				fields: ['typed', 'found', 'in_text?', 'times occouring'],
				items: [],
				text: 'insignificant little blue green planet',
				phrase: '',
				phraseList: [],
				occouring_phrases: [],
				paragraph_text: txt,
				styled_paragraph_text: txt,
				score_scaling: 2 //odprilike koliko gresaka po rijeci u recenici moze biti
			}
		},
	methods: {
		addToPhraseList(inputed_phrase) {
			let phrase = this.cleanInputText(inputed_phrase)
			var includes = false
			for (let i = 0; i < this.phraseList.length; i++) {
				if (this.phraseList[i].phrase == phrase) {
					includes = true
				}
			}

			if (phrase == '' || includes) {
				alert("Input is empty or already included entered!")
			} else {	
				this.phraseList.push({ "phrase": phrase })
				let found = this.search(phrase)
				if(found != "") {
					console.log(found)
					this.items.push({ "typed": phrase,"found": found, "in_text?": "Yes", "times occouring": this.count_word_occ(this.paragraph_text, found, true)})
					//hajlajtuj tekst u paragrafu koji ce biti prikazan
					this.highlightText()
				} else {
					this.items.push({"typed": phrase, "in_text?": "No"})
				}
			}


		},
		search(text) {
			let p = this.paragraph_text.split(" ");
			let t = text.split(" ");

			let found = ""
			for (let i = 0;  i < p.length; i++) {
				var dist = 0
				let current_phrase_in_text = p.slice(i, i + t.length)
				dist = this.levensthein(current_phrase_in_text, t)
				// ne znam koji bi bio najoptimalniji score pa sam stavio da mozemo podesavati u odnosu na broj rijeci u recenici
				if (dist < t.length * this.score_scaling) {
					found = current_phrase_in_text.join(" ")
					
					this.styled_paragraph_text = this.paragraph_text
					break;
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
		},
		cleanInputText(input) {
			let input_l = input.trim().split("") // skinemo sve razmake sa pocetka i kraja i podijelimo recenicu u listu slova
			let output_l = []
			let found = false; // bool koji oznacava da li smo naisli na razmak
			for (let i = 0; i < input_l.length; i++) { //prolazimo kroz sva slova u recenici
				if (input_l[i] == " ") { //ako smo naisli na razmak
					if (!found) { // i nije vise od jedan razmak
						output_l.push(input_l[i]) //dodaj razmak u listu
						found = true // podesimo da smo ga nasli, svaki sledeci posle njega ce biti skipovan
					}
				} else { //ako smo naisli na slovo
					output_l.push(input_l[i]) //dodamo slovo u listu
					found = false // ako je bio razmak prije, vatimo ga na false za nastavak
				}
			}
			return output_l.join("")
		},
		highlightText() {
			let style_text = this.paragraph_text

			for (let i = 0; i < this.items.length; i++) {
				let phrase = this.items[i].found
				let stylzed = "<strong style=\"text-decoration: underline; color: pink;\">" + phrase + "</strong>"
				style_text = style_text.replaceAll(phrase, stylzed)
			}
			this.styled_paragraph_text = style_text
		},
		
		count_word_occ(string, subString, allowOverlapping) {
			if (subString.length <= 0) return (string.length + 1);

			var n = 0,
				pos = 0,
				step = allowOverlapping ? 1 : subString.length;

			while (pos >= 0) {
				pos = string.indexOf(subString, pos);
				if (pos >= 0) {
						n++;
						pos += step;
				} else break;
			}
			return n;
		}
	}
}
</script>

<style>
	#app {
		/* font-family: Avenir, Helvetica, Arial, sans-serif; */
		font-family: 'Courier New', monospace;
		-webkit-font-smoothing: antialiased;
		-moz-osx-font-smoothing: grayscale;
		text-align: center;
		color: #cacaca;
		margin-top: 60px;
	}
</style>
