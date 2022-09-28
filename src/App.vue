<template>
	<div class="min-h-[calc(100vh-100px)]">
		<div class="grid grid-cols-1 gap-1 p-14 pb-4 max-w-[1200px]">
			<div class="week">
				<div>/</div>
				<div>Sunday</div>
				<div>Monday</div>
				<div>Tuesday</div>
				<div>Wednesday</div>
				<div>Thursday</div>
				<div>Friday</div>
				<div>Saturday</div>
			</div>
			<div v-for="(week, i) in table" :key="week" class="week">
				<div class="relative">
					<input type="text" maxlength="5" v-model="week.start" class="absolute top-0 right-0 text-sm">
					<input type="text" maxlength="5" v-model="week.finish" class="absolute bottom-0 right-0 text-sm">
				</div>
				<div v-for="(day, j) in week.days" :key="day" class="day">
					<input type="text" :id="`${i}:${j}`" v-model="day.val" @input="dayInputChanged">
				</div>
			</div>
		</div>
		<div class="flex items-center justify-between mx-14">
			<div class="flex items-center justify-start gap-5">
				<button @click="addRow">+ add row</button>
				<button @click="deleteRow">- delete row</button>
				<label for="apply-row">
					<input type="checkbox" id="apply-row" v-model="ctrls.applyRow">
					<span>apply row</span>
				</label>
				<label for="apply-col">
					<input type="checkbox" id="apply-col" v-model="ctrls.applyCol">
					<span>apply column</span>
				</label>
				<button @click="clearAll">x clear all</button>
			</div>
			<button class="download" @click="createPDF">download PDF</button>
		</div>
	</div>

	<footer class="text-center m-8">
		Made with <span>Vue.js</span> by <a href="https://github.com/orhanemree">orhanemree</a>.
		Open Source on <a href="https://github.com/orhanemree/proplanner">GitHub</a>.
	</footer>
</template>

<script>
	// hey: so many for loop alert!

	import { jsPDF } from "jspdf";
	export default {
		data() {
			return {
				ctrls: { applyRow: false, applyCol: false },
				table: []
			}
		},
		mounted() {
			const table = new Array(6);

			for (let i = 0; i < table.length; ++i) {
				table[i] = { days: new Array(7), start: "xx.xx", finish: "xx.xx" };
				for (let j = 0; j < table[i].days.length; ++j) {
					table[i].days[j] = { val: "" };
				}
			}
			this.table = table;
		},
		methods: {
			dayInputChanged(e) {
				const id = e.target.id.split(":");
				
				if (this.ctrls.applyRow) {
					for (let  i = 0; i < this.table[id[0]].days.length; ++i){
						this.table[id[0]].days[i].val = e.target.value.trim();
					}
				}

				if (this.ctrls.applyCol) {
					for (let i = 0; i < this.table.length; ++i){
						this.table[i].days[id[1]].val = e.target.value.trim();
					}
				}

				this.table[id[0]].days[id[1]].val = e.target.value.trim();
			},
			addRow() {
				const row = { days: new Array(7), start: "xx.xx", finish: "xx.xx" };
				for (let i = 0; i < row.days.length; ++i) {
					row.days[i] = { val: "" };
				}
				this.table.push(row);
			},
			deleteRow() {
				this.table.pop();
			},
			clearAll() {
				for (let i = 0; i < this.table.length; ++i) {
					this.table[i].start = "xx.xx";
					this.table[i].finish = "xx.xx";
					for (let j = 0; j < this.table[i].days.length; ++j) {
						this.table[i].days[j] = { val: "" };
					}
				}
			},
			createPDF() {
				const doc = new jsPDF({ orientation: "landscape" });
				doc.setFontSize(8);
				doc.text(new Date().toLocaleString(), 5, 5);
				doc.text("proplanner.vercel.app by orhanemree", parseInt(doc.internal.pageSize.getWidth()/2-10), parseInt(doc.internal.pageSize.getHeight()-10));
				doc.setFontSize(12);

				const chars = { "ı": "i", "ğ": "g", "ü": "u", "ş": "s", "ö": "o", "ç": "c" };
				const days = ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"];

				console.log(this.table);

				for (let i = 0; i < this.table.length; ++i) {
					if (i !== 0 && i%6 === 0) {
						doc.addPage("a4", "landscape");
						doc.setFontSize(8);
						doc.text(new Date().toLocaleString(), 5, 5);
						doc.text("proplanner.vercel.app by orhanemree", parseInt(doc.internal.pageSize.getWidth()/2-10), parseInt(doc.internal.pageSize.getHeight()-10));
						doc.setFontSize(12);
					}
					
					const y = (i%6+1)*26;
					
					doc.text(this.table[i].start, 22, y + 5);
					doc.text(this.table[i].finish, 22, y + 22);
					for (let j = 0; j < this.table[i].days.length; ++j) {
						const x = (j+1)*34;

						doc.text(days[j], x+16-days[j].length, 20)
						doc.rect(x, y, 33, 25);
						const text = this.table[i].days[j].val.toLowerCase();
						doc.text(text.replace(/[ığüşöç]/g, m => chars[m]), x+16-text.length, y+15);
					}
				}

				doc.save("proplanned.pdf");
			}
		}
	}
</script>