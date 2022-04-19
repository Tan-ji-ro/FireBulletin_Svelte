<script>
	// Imports
	import { initializeApp } from "firebase/app";
	import {
		getFirestore,
		collection,
		query,
		onSnapshot,
		doc,
		setDoc,
		deleteDoc,
		updateDoc,
		where,
	} from "firebase/firestore";
	import {
		getStorage,
		ref as ref_storage,
		uploadBytes,
		getDownloadURL,
	} from "firebase/storage";
	import {
		getDatabase,
		set,
		onValue,
		ref as ref_database,
	} from "firebase/database";
	import { v4 as uuidv4 } from "uuid";
	import {
		Col,
		Container,
		Row,
		Styles,
		Accordion,
		AccordionItem,
		Button,
		FormGroup,
		InputGroup,
		InputGroupText,
		Input,
		Navbar,
		NavbarBrand,
		Nav,
		NavItem,
		NavLink,
		Carousel,
		CarouselControl,
		CarouselIndicators,
		CarouselItem,
		CarouselCaption,
	} from "sveltestrap";
	import Search from "svelte-search";

	// Firebase initialization
	initializeApp({
		apiKey: "AIzaSyCDiiWGodZMH2Ey8J9n_oWPaylc4I8pxaA",
		authDomain: "firenews-server.firebaseapp.com",
		projectId: "firenews-server",
		storageBucket: "firenews-server.appspot.com",
		messagingSenderId: "724529920072",
		appId: "1:724529920072:web:2d33fc57cfa27367a3407d",
		measurementId: "G-2MBC3JXTKK",
		databaseURL:
			"https://firenews-server-default-rtdb.asia-southeast1.firebasedatabase.app/",
	});

	// Variables for main handler
	const fs_db = getFirestore();
	const storage = getStorage();
	const rt_db = getDatabase();

	let element_firestore = [];

	let uuid = uuidv4();
	let newNewsTitleText = "";
	let newNewsSummaryText = "";
	let newNewsDetailText = "";
	let newDate = new Date().toString();
	let search_value = "";

	// Link handler
	let newLink = "";
	let newComment = "";
	const dbRef = ref_database(rt_db, "Highlights/");

	// Validation handler
	let hasBeenClicked = false;
	let clickval = false;

	$: isValidNewsTitle = newNewsTitleText.length > 0;
	$: isValidNewsSummary = newNewsSummaryText.length > 0;
	$: isValidNewsDetail = newNewsDetailText.length > 0;

	$: isValidLink = newLink.length > 0;
	$: isValidComment = newComment.length > 0;

	// Carousel handler
	let items = [];
	let activeIndex = 0;

	// List print
	onSnapshot(query(collection(fs_db, "NewsList")), (querySnapshot) => {
		const news_line = [];

		querySnapshot.forEach((doc) => {
			news_line.push(doc.data());
		});

		element_firestore = news_line;
	});

	// Add item
	function addFirestoreItem() {
		const selectedFile = document.getElementById("articleFile").files[0];
		const name = +new Date() + "-" + selectedFile.name;
		const picref = ref_storage(storage, `Article/${name}`);

		uploadBytes(picref, selectedFile).then((snapshot) => {
			console.log(
				"Uploaded a blob or file! The file is: " +
					JSON.stringify(snapshot)
			);

			getDownloadURL(picref).then((url) => {
				const docRef = setDoc(doc(fs_db, "NewsList", uuid), {
					newsId: uuid,
					newsTitle: newNewsTitleText,
					newsSummary: newNewsSummaryText,
					newsDetail: newNewsDetailText,
					newsDate: newDate,
					newsPicture: url,
				});
				console.log("Document written with ID: ", docRef.newsId);
			});
		});
	}

	// Delete item
	function deleteFirestoreItem(itemId) {
		if (confirm("Are you sure you are deleting a news?") == true) {
			deleteDoc(doc(fs_db, "NewsList", itemId));
			console.log("Confirmed");
		} else {
			console.log("Cancelled");
		}
	}

	// Update items
	function updateFirestoreItem(
		itemId,
		newNewsTitleValue,
		newNewsSummaryValue,
		newNewsDetailValue
	) {
		updateDoc(doc(fs_db, "NewsList", itemId), {
			newsTitle: newNewsTitleValue,
			newsSummary: newNewsSummaryValue,
			newsDetail: newNewsDetailValue,
		});
	}

	// Validation
	function handleSubmission() {
		hasBeenClicked = true;

		if (isValidNewsTitle && isValidNewsSummary && isValidNewsDetail) {
			addFirestoreItem();
		}
	}

	function DatabaseValidation() {
		clickval = true;

		if (isValidLink && isValidComment) {
			DataImport();
		}
	}

	// Topic search
	function TopicSearch() {
		if (search_value.length > 0) {
			const q_s = query(
				collection(fs_db, "NewsList"),
				where("newsTitle", "==", search_value)
			);

			onSnapshot(q_s, (querySnapshot) => {
				const article = [];

				querySnapshot.forEach((doc) => {
					article.push(doc.data());
				});

				element_firestore = article;
			});
		} else {
			onSnapshot(
				query(collection(fs_db, "NewsList")),
				(querySnapshot) => {
					const article = [];

					querySnapshot.forEach((doc) => {
						article.push(doc.data());
					});
					element_firestore = article;
				}
			);
		}
	}

	// Database import
	function DataImport() {
		const unix = new Date().getTime();

		set(ref_database(rt_db, "Highlights/" + unix), {
			picture_link: newLink,
			comment: newComment,
		});
	}

	// Database export
	onValue(
		dbRef,
		(snapshot) => {
			snapshot.forEach((childSnapshot) => {
				items = [...items, childSnapshot.val()];
			});
		},
		{
			onlyOnce: true,
		}
	);
</script>

<!-- Main UI -->

<main>
	<Styles />
	<Navbar light expand="md">
		<NavbarBrand>Firebase News Bulletin with Svelte</NavbarBrand>
		<Nav class="ms-auto" navbar>
			<NavItem>
				<NavLink href="#">Homepage</NavLink>
			</NavItem>
			<NavItem>
				<NavLink href="#">About</NavLink>
			</NavItem>
		</Nav>
	</Navbar>
	<br />
	<Container>
		<Row cols={2}>
			<Col>
				<h2>Firestore news list</h2>
				<!-- Accordion list and modify form -->
				<Search
					label="My search field"
					hideLabel
					bind:value={search_value}
					placeholder="Search your topic here"
					on:submit={TopicSearch}
				/>
				<br />
				<Accordion flush stayOpen>
					{#each element_firestore as firestoreListItem}
						<AccordionItem header={firestoreListItem.newsTitle}>
							<h6>Date written: {firestoreListItem.newsDate}</h6>
							<br />
							{firestoreListItem.newsSummary}
							<br />
							<br />
							{firestoreListItem.newsDetail}
							<br />
							<br />
							<img
								style="width: 320px, height: 480px"
								src={firestoreListItem.newsPicture}
								alt="Article pic"
							/>
							<br />
							<br />
							<Button
								on:click={() =>
									deleteFirestoreItem(
										firestoreListItem.newsId
									)}>Delete</Button
							>
							{#if firestoreListItem.editMode == true}
								<form>
									<br />
									<textarea
										cols="75"
										placeholder="News title"
										value={firestoreListItem.newsTitle}
										bind:this={firestoreListItem.inputUpdate1}
									/>
									<br />
									<textarea
										placeholder="News summary"
										rows="5"
										cols="75"
										value={firestoreListItem.newsSummary}
										bind:this={firestoreListItem.inputUpdate2}
									/>
									<br />
									<textarea
										placeholder="News detail"
										rows="15"
										cols="75"
										value={firestoreListItem.newsDetail}
										bind:this={firestoreListItem.inputUpdate3}
									/>
									<br />
								</form>
								<Button
									on:click={() => {
										updateFirestoreItem(
											firestoreListItem.newsId,
											firestoreListItem.inputUpdate1
												.value,
											firestoreListItem.inputUpdate2
												.value,
											firestoreListItem.inputUpdate3.value
										);
									}}>Modify</Button
								>
							{:else}
								<Button
									on:click={() => {
										firestoreListItem.editMode = true;
									}}>Modify</Button
								>
							{/if}
						</AccordionItem>
					{/each}
				</Accordion>
			</Col>

			<Col>
				<h2>Add news form</h2>
				<!-- New article form -->
				<FormGroup>
					<InputGroup>
						<InputGroupText>News title</InputGroupText>
						<Input
							type="text"
							placeholder="News title"
							bind:value={newNewsTitleText}
						/>
						{#if hasBeenClicked && !isValidNewsTitle}
							<br />
							<p class="validation-error">
								Please enter a name for your news
							</p>
						{/if}
					</InputGroup>
					<br />
					<InputGroup>
						<InputGroupText>News summary</InputGroupText>
						<Input
							type="textarea"
							placeholder="News summary"
							bind:value={newNewsSummaryText}
						/>
						{#if hasBeenClicked && !isValidNewsSummary}
							<br />
							<p class="validation-error">
								Please enter a summary for your news
							</p>
						{/if}
					</InputGroup>
					<br />
					<InputGroup>
						<InputGroupText>News detail</InputGroupText>
						<Input
							type="textarea"
							placeholder="News detail"
							bind:value={newNewsDetailText}
						/>
						{#if hasBeenClicked && !isValidNewsDetail}
							<br />
							<p class="validation-error">
								Please enter a detail for your news
							</p>
						{/if}
					</InputGroup>
					<br />
					<InputGroup>
						<InputGroupText>News picture</InputGroupText>
						<Input type="file" name="file" id="articleFile" />
					</InputGroup>
				</FormGroup>
				<br />
				<Button on:click={handleSubmission}>Submit your news</Button>
			</Col>
		</Row>

		<br />
		<br />
		<br />

		<Row cols={2}>
			<Col>
				<h2>Highlight carousel</h2>
				<Carousel dark {items} bind:activeIndex>
					<CarouselIndicators bind:activeIndex {items} />

					<div class="carousel-inner">
						{#each items as item, index}
							<CarouselItem bind:activeIndex itemIndex={index}>
								<img
									src={item.picture_link}
									class="d-block w-100"
									alt={`${item} ${index + 1}`}
								/>
								<CarouselCaption captionText={item.comment} />
							</CarouselItem>
						{/each}
					</div>

					<CarouselControl
						direction="prev"
						bind:activeIndex
						{items}
					/>
					<CarouselControl
						direction="next"
						bind:activeIndex
						{items}
					/>
				</Carousel>
			</Col>

			<Col>
				<h2>Carousel import</h2>
				<FormGroup>
					<InputGroup>
						<InputGroupText>Picture link</InputGroupText>
						<Input
							placeholder="Picture link"
							bind:value={newLink}
						/>
						{#if clickval && !isValidLink}
							<br />
							<p class="validation-error">
								Please enter a link for your highlight
							</p>
						{/if}
					</InputGroup>
					<br />
					<InputGroup>
						<InputGroupText>Comments</InputGroupText>
						<Input placeholder="Comments" bind:value={newComment} />
						{#if clickval && !isValidComment}
							<br />
							<p class="validation-error">
								Please enter a comment for your highlight
							</p>
						{/if}
					</InputGroup>
				</FormGroup>
				<br />
				<Button on:click={DatabaseValidation}
					>Submit your highlight</Button
				>
			</Col>
		</Row>
	</Container>
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h2 {
		color: #ff3e00;
		text-align: center;
	}

	img {
		width: 320px;
	}

	:global([data-svelte-search] input) {
		width: 100%;
		padding-top: 2.5px;
		padding-bottom: 2.5px;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}

		.validation-error {
			color: red;
			margin-top: 5px;
			width: 100%;
		}
	}
</style>
