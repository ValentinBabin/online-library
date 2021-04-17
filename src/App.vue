<template>
	<h1>Online Library</h1>
	<form @submit.prevent="addBook()">
		<label for="nameBook">Nom</label>
		<input
			v-model="newBook.nameBook"
			id="nameBook"
			name="nameBook"
			autocomplete="off"
		>
		<label for="writerBook">Auteur(s)</label>
		<input
			v-model="newBook.writerBook"
			id="writerBook"
			name="writerBook"
			autocomplete="off"
		>
		<button>Ajouter le livre</button>
	</form>
	<h2>Listes des livres</h2>
	<ul>
		<li v-for="(book, index) in books" :key="index">
			<div class="content">
				<div class="title" :class="{ have: book.have }" @click="doneBook(book)" >
					<div>{{ book.nameBook }}</div>
					<small>{{ book.writerBook }}</small>
				</div>
				<div class="editing" v-if="book.isEditing">
					<form @submit.prevent="updateBook(index)">
						<label for="readedPages">Pages lus</label>
						<input
							v-model.number="book.readedPages"
							type="number"
							id="readedPages"
							name="readedPages"
							autocomplete="off"
						>
						<label for="totalPages">Pages totales</label>
						<input
							v-model.number="book.totalPages"
							type="number"
							id="totalPages"
							name="totalPages"
							autocomplete="off"
						>
						<button>Modifier</button>
					</form>
				</div>
			</div>
			<div class="actions">
				<button v-if="!book.isEditing && book.have" @click="updateBook(index)">Modifier</button>
				<button @click="removeBook(index)">Enlever</button>
			</div>
			
			<div v-if="book.readingPercent > 0 && book.readingPercent < 100 && book.have"
				v-bind:style="{ width: book.readingPercent + '%' }" class="progress-bar">
				{{book.readingPercent}}%
			</div>
			<div v-if="book.readingPercent === 100 && book.have" 
				v-bind:style="{ width: book.readingPercent + '%' }" class="progress-bar-total">
				{{book.readingPercent}}%
			</div>
		</li>
	</ul>
	<h4 v-if="books.length === 0">Liste vide.</h4>
</template>

<script>
	import { ref } from 'vue';
	export default {
		name: 'App',
		mounted() {
			this.saveData();
		},
		setup () {
			const newBook = { nameBook:'', writerBook:''};
			const defaultData = [{
				have: true,
				isEditing: false,
				readedPages: 0,
				totalPages: 0,
				readingPercent: 0,
				nameBook: 'Nom du livre',
				writerBook: 'Auteur(s) du livre'
			}];
			const bookData = JSON.parse(localStorage.getItem('books')) || defaultData;
			const books = ref(bookData);

			function addBook () {
				if (newBook.nameBook && newBook.writerBook) {
					books.value.push({
						have: false,
						isEditing: false,
						readedPages: 0,
						totalPages: 0,
						readingPercent: 0,
						nameBook: newBook.nameBook,
						writerBook: newBook.writerBook
					});
					books.value.sort(compateToSort);
					newBook.nameBook = '';
					newBook.writerBook = '';
				}
				saveData();
			}

			function updateBook (index) {
				if (books.value[index].isEditing === true) {
					if (books.value[index].readedPages && books.value[index].totalPages) {
						books.value.splice(index, 1, {
							have: books.value[index].have,
							isEditing: false,
							readedPages: books.value[index].readedPages,
							totalPages: books.value[index].totalPages,
							readingPercent: Math.round(( (books.value[index].readedPages * 100)/ books.value[index].totalPages ) * 100) / 100,
							nameBook: books.value[index].nameBook,
							writerBook: books.value[index].writerBook
						});
						// books.value[index].readedPages = '';
						// books.value[index].totalPages = '';
						books.value.sort(compateToSort);
					}
					books.value[index].isEditing = false;
					saveData();
				} else {
					books.value[index].isEditing = true;
				}
			}

			function doneBook (book) {
				if (!book.isEditing) {
					book.have = !book.have
				}
				saveData();
			}

			function removeBook (index) {
				books.value.splice(index, 1);
				books.value.sort(compateToSort);
				saveData();
			}

			function compateToSort(x, y) {
				if (x.nameBook < y.nameBook) {
					return -1;
				}
				if (x.nameBook > y.nameBook) {
					return 1;
				}
				return 0
			}

			function saveData () {
				const storageData = JSON.stringify(books.value);
				localStorage.setItem('books', storageData);
			}

			return {
				books,
				newBook,
				addBook,
				updateBook,
				doneBook,
				removeBook,
				compateToSort,
				saveData
			}
		}
	}
</script>

<style lang="scss">
@import url('https://fonts.googleapis.com/css2?family=Raleway:wght@300;400;500;600;700;800&display=swap');
@import url('https://fonts.googleapis.com/css2?family=Karla:wght@300;400;500;600;700;800&display=swap');
$border: 2px solid
	rgba(
		$color:  #201F3D,
		$alpha: 0.35,
	);
$size1: 6px;
$size2: 12px;
$size3: 18px;
$size4: 24px;
$size4-5: 35px;
$size5: 48px;
$textColor: #201F3D;
$primaryColor: #EE544B;
$secondTextColor: #F3F5FB;
$backgroundColor: $secondTextColor;
$font: 'Karla', 'Raleway', Arial, sans-serif;
body {
	margin: 0;
	padding: 0;
	font-family: $font;
	letter-spacing: -1.5px;
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
	background-color: $backgroundColor;
	color: $textColor;
	#app {
		max-width: 600px;
		margin-left: auto;
		margin-right: auto;
		padding: 20px;
		h1 {
			font-weight: bold;
			font-size: $size4-5;
			text-align: left;
			margin-top: 60px;
		}
		form {
			display: flex;
			flex-direction: column;
			width: 100%;
			label {
				font-size: 14px;
				font-weight: bold;
			}
			input,
			button {
				height: $size5;
				box-shadow: none;
				outline: none;
				padding-left: $size2;
				padding-right: $size2;
				border-radius: $size1;
				font-family: $font;
				font-size: 18px;
				margin-top: $size1;
				margin-bottom: $size2;
			}
			input {
				background-color: transparent;
				border: $border;
				color: inherit;
			}
		}
		button {
			cursor: pointer;
			background-color: $primaryColor;
			border: 1px solid $primaryColor;
			color: $secondTextColor;
			font-weight: bold;
			outline: none;
			border-radius: $size1;
		}
		h2 {
			font-size: 22px;
			border-bottom: $border;
			padding-bottom: $size1;
		}
		ul {
			padding: 10px 0;
			li {
				justify-content: space-between;
				align-items: center;
				border: $border;
				padding: $size2 $size3 $size2+6px $size3;
				border-radius: $size1;
				margin-bottom: $size2;
				display: grid;
				grid-template-columns: repeat(12, 1fr);
				grid-gap: 0;
				position: relative;
				div.content{
					grid-column: 1/8;
					grid-row: 1;
					padding-right: 10px;

					div.title {
						cursor: pointer;
						div{
							font-size: $size3;
							margin-bottom: 5px;
						}
						small{
							font-size: $size2;
						}
					}
					.have {
						font-weight: bold;
					}
					div.editing{
						form {
						display: flex;
						flex-direction: column;
						width: 100%;
						margin-top: 10px;
							input {
								width: 100%;
								box-sizing: border-box;
								box-shadow: none;
								outline: none;
								padding-left: $size2;
								padding-right: $size2;
								border-radius: $size1;
								font-size: $size3;
								margin-top: $size1;
								margin-bottom: $size2;
								background-color: transparent;
								border: $border;
								color: inherit;
							}
							button {
								height: auto;
								font-family: $font;
								font-size: $size2;
								padding: $size1;
								font-weight: 600;
								margin: 0 2px;
							}
							label{
								font-size: $size2;
							}
						}
					}
				}
				div.actions{
					grid-column: 8/13;
					grid-row: 1;
					display: flex;
					flex-direction: row;
					align-content: center;
					justify-content: flex-end;
					button {
						font-family: $font;
						font-size: $size2;
						padding: $size1;
						font-weight: 600;
						margin: 0 2px;
						&:first-child{
							margin-left: 0;
						}
						&:last-child{
							margin-right: 0;
						}
					}
				}

				div.progress-bar{
					position: absolute;
					left: 0;
					bottom: 0;
					height: 6px;
					background: $primaryColor;
					border-radius: 0 0 0 4px;
					display: flex;
					flex-direction: column;
					align-items: center;
					justify-content: center;
					&-total{
						@extend .progress-bar;
						border-radius: 0 0 4px 4px;
					}
				}
			}
		}
		h4 {
			text-align: center;
			opacity: 0.5;
			margin: 0;
		}
	}
}
</style>
