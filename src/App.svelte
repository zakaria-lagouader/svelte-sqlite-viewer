<script>
	import Navbar from "./components/Navbar.svelte";
	import Table from "./Table.svelte";

	const worker = new Worker('./worker.sql-wasm.js');
	worker.onerror = e => console.log("Worker error: ", e);

	// state
	let editorText = "";
	let tableData = null;
	let fileInput = null;
	let tablesNames = [];
	let error = null;

	// methodes
	const run = () => {
		worker.onmessage = event => {
			const { results } = event.data;
			if (!results) {
				console.log({message: event.data.error});
				error = event.data.error;
				return;
			}else {
				error = null;
			}
			console.log("Running SQL Query");
			console.log(results); // The result of the query
			tableData = results[0];
		};
		worker.postMessage({
			action: "exec",
			sql: editorText
		});
	}

	const uploadFile = (e) => {
		const file = e.target.files[0];
		const reader = new FileReader();
		reader.onload = () => {
			worker.onmessage = function () {
				console.log("File uploaded");
				editorText = "SELECT `name`  FROM `sqlite_master` WHERE type='table';";
				getTablesNames();
			};
			try {
				worker.postMessage({ action: 'open', buffer: reader.result }, [reader.result]);
			}
			catch (exception) {
				worker.postMessage({ action: 'open', buffer: reader.result });
			}
		}
		reader.readAsArrayBuffer(file);
	}

	const getTablesNames = () => {
		worker.onmessage = event => {
			const { results } = event.data;
			if (!results) {
				console.log({message: event.data.error});
				return;
			}
			tablesNames = results[0].values.flat();
		};
		worker.postMessage({
			action: "exec",
			sql: "SELECT `name`  FROM `sqlite_master` WHERE type='table';"
		});
	}

	const showTable = (name) => {
		editorText = `SELECT * FROM ${name}`
		run();
	}

	const clear = () => {
		editorText = "";
	}

</script>

<Navbar />

<section id="showcase" class="py-5">
	<div class="container">
		<div class="mb-5">
			<h1 class="text-center display-4 mb-4">SQLie Online</h1>
			{#if error}
				<div class="alert alert-danger my-3" role="alert">
					{error}
				</div>
			{/if}
			<div class="form-group">
				<textarea 
					class="form-control" 
					bind:value={editorText} 
					rows="10"
					on:input={() => error = null}
				></textarea>
			</div>
			<button class="btn btn-primary" on:click={run}>Run</button>
			<button class="btn btn-success" on:click={() => fileInput.click()}>Upload</button>
			<button class="btn btn-danger" on:click={clear}>Clear</button>
			<input type="file" style="display: none;" bind:this={fileInput} on:change={uploadFile}>
		</div>
		<div class="row">
			<div class="col-md-3">
				<ul class="list-group">
					{#each tablesNames as name}
						<button 
							type="button" 
							class="list-group-item list-group-item-action"
							on:click={() => showTable(name)}
						>
							{name}
						</button>
					{/each}
				</ul>
			</div>
			<div class="col-md-9 mx-auto">
				{#if tableData}
					<Table headers={tableData.columns} rows={tableData.values} />
				{:else}
					<h5 class="text-center my-4">Table empty</h5>
				{/if}
			</div>
		</div>
	</div>
</section>