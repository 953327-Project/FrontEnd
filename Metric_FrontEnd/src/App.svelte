<script>
	import { onMount } from "svelte";
  
	const apiURL = "http://127.0.0.1:5000/get_score_csv";

	let projects = [];
	let selected;
	  
	onMount(async function() {
		const response = await fetch(apiURL);
		projects = await response.json();

		// sort array project by score.
		projects.sort(function(a, b){
			return b.total_score - a.total_score;
		})
	});
  
	  
	function deleteProject(index){
		let tempIndex = 0;

		if(index == null){
			projects.slice(tempIndex, 1);
		}else{
			projects.splice(index, 1);
		}

		projects = projects;

		//console.log(proejcts, index)
	}

	function handleSubmit() {
		let index = selected;
	
		deleteProject(index);
	}
	
  </script>

<!-- HTML -->
  
	<div class="container-fluid">
		<h2 class="display-4 mb-4">API builder Ranking</h2>	

		<table class="table table-bordered">
			<thead>
				<tr>
					<th>🏆#</th>
					<th>Project Name</th>
					<th>Total score</th>
				</tr>
			</thead>
			<tbody>
				{#if projects.length}
						{#each projects as project, index}	
							<tr>
								<td>{index+1}</td>
								<td>{project.project_name}</td>
								<td>{project.total_score}</td>
							</tr>
						{/each}
				{:else}
					<tr>
						<td colspan="5">There are no other project</td>
					</tr>
				{/if}
			</tbody>
		</table>

	{#if projects.length}
		<form on:submit|preventDefault={handleSubmit}>
			<select bind:value={selected}>
				{#each projects as project, index}
					<option value={index}>
						{project.project_name}
					</option>
				{/each}
				<option value="x">
				</option>
			</select>

			<button disabled={selected == "x"} type="submit"> Remove </button>
		</form>
	{:else}
		<button on:click={ async () => {
			const response = await fetch(apiURL);
			projects = await response.json();

			// sort array project by score.
			projects.sort(function(a, b){
				return b.total_score - a.total_score;
			})
		}}>Refresh</button>
	{/if}

	<hr>

	<p style="opacity: 50%;">Total score calculated by each category, The highest score in the category got 4 point to lowest got 1 point.</p>
  	
	<table class="table table-bordered">
		<thead>
			<tr>
				<th>Project Name</th>
				<th>Code Comprehen</th>
				<th>Doc Comprehen</th>
				<th>Code Reusability</th>
				<th>Built Reusability</th>
				<th>Test Quality</th>
				<th>Team Activeness</th>
			</tr>
		</thead>
		<tbody>
			{#if projects.length}
					{#each projects as project}	
						<tr>
							<td>{project.project_name}</td>
							<td>{project.code_comprehen}</td>
							<td>{project.doc_comprehen}</td>
							<td>{project.code_reusability}</td>
							<td>{project.built_reusability}</td>
							<td>{project.test_quality}</td>
							<td>{project.team_activeness}</td>
						</tr>
					{/each}
			{:else}
				<tr>
					<td colspan="7">There are no other project</td>
				</tr>
			{/if}
		</tbody>
	</table>

  </div>
  <link href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.0/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-9aIt2nRpC12Uk9gS9baDl411NQApFmC26EwAOH8WgZl5MYYxFfc+NcPb1dKGj7Sk" crossorigin="anonymous">