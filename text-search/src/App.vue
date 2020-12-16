<template>
	<div id="app">
		<b-container>
			<h2 style="text-weight: bold; color: pink; font-weight: bold;">Search text</h2>
			<!-- <h5>With having fat-fingers like ćevapčići </h5> -->

			
			<b-input-group prepend="Phrase:" class="mt-3">
				<b-form-input style="background: lightgray;" v-model="phrase" v-on:keyup.enter="addToPhraseList(phrase)"></b-form-input>
				<b-input-group-append>
					<b-button variant="success" @click="addToPhraseList(phrase)">Search</b-button>
				</b-input-group-append>
			</b-input-group>

			<p ref="para" v-html="styled_paragraph_text" class="mt-4">
				{{styled_paragraph_text}}
			</p>

			
			<b-table small striped class="mt-5" hover :items="phraseList" :fields="fields" style="color: lightgrey"></b-table>
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
				text: 'insignificant little blue green planet',
				phrase: '',
				phraseList: [],
				paragraph_text: txt,
				styled_paragraph_text: txt,
				score_scaling: 2 //odprilike gledano, koliko gresaka po rijeci u recenici moze biti
			}
		},
	methods: {
		addToPhraseList(inputed_phrase) {
			let phrase = this.cleanInputText(inputed_phrase)
			var includes = false //da li se unesena fraza vec nalazi u listi
			for (let i = 0; i < this.phraseList.length; i++) {
				if (this.phraseList[i].typed == phrase) { //check za gore pomenutu vrijednost
					includes = true
				}
			}

			if (phrase == '' || includes) {
				alert("Input is empty or already typed in!")
			} else {	
				let found = this.search(phrase)
				if(found != "") {
					this.phraseList.push({ "typed": phrase,"found": found, "in_text?": "Yes", "times occouring": this.count_word_occ(this.paragraph_text, found, false)})
					this.highlightText()
				} else {
					this.phraseList.push({"typed": phrase, "in_text?": "No"})
				}
			}


		},
		search(text) {
			let found_word = "" // nadjena rijec

			if (this.paragraph_text.search(text) != -1) { //ako je nasao tacno tu koju smo unijeli, ne radimo levenstajn
				found_word = text
				return found_word
			} else {
				let p = this.paragraph_text.split(" ");
				let t = text.split(" ");
				for (let i = 0;  i < p.length; i++) {
					var n_operations = 0 // slicnost ukucane rijeci i trenutne kroz koju prolazimo, odnosno broj operacija (add, remove delete) koje se trebaju izvrsiti da bi bile iste
					let current_phrase_in_text = p.slice(i, i + t.length)
					n_operations = this.levensthein(current_phrase_in_text.join(" "), t.join(" "))
					if (n_operations < t.length * this.score_scaling) {// ne znam koji bi bio najoptimalniji score pa sam stavio da mozemo podesavati u odnosu na broj rijeci u unesenom tekstu
						found_word = current_phrase_in_text.join(" ")
						
						this.styled_paragraph_text = this.paragraph_text
						break;
					}
				}
				return found_word
			}

		},
		levensthein(s1, s2) {
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

			for (let i = 0; i < this.phraseList.length; i++) {
				let phrase = this.phraseList[i].found
				let stylzed = "<strong style=\"text-decoration: underline; color: pink;\">" + phrase + "</strong>"
				style_text = style_text.replaceAll(phrase, stylzed)
			}
			this.styled_paragraph_text = style_text
		},
		count_word_occ(string, subString, allowOverlapping) {
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
