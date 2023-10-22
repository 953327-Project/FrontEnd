<script>
	import { onMount } from "svelte";
	// @ts-ignore
	import FusionCharts from "fusioncharts";
	// @ts-ignore
  	import Charts from "fusioncharts/fusioncharts.charts";
	// @ts-ignore
  	import FusionTheme from "fusioncharts/themes/fusioncharts.theme.fusion";
	// @ts-ignore
  	import { fcRoot } from "svelte-fusioncharts";
	import Footer from "./lib/Footer.svelte"
	// @ts-ignore
  	fcRoot(FusionCharts, Charts, FusionTheme);
	
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

		chartRander(projects)
	});

	function chartRander(projectArray) {
			const chartData = projectArray.map(item => ({
			label: item.project_name,
			code_comprehen: item.code_comprehen,
			doc_comprehen: item.doc_comprehen,
			code_reusability: item.code_reusability,
			built_reusability: item.built_reusability,
			test_quality: item.test_quality,
			team_activeness: item.team_activeness
		}));

		const dataSource = {
		chart: {
			caption: "Project Metrics Chart",
			subcaption: "Comparison of Various Attributes",
			xaxisname: "Project Name",
			yaxisname: "Value",
			theme: "fusion",
			showValues: "1",
		},
		categories: [
			{
			category: chartData.map(item => ({ label: item.label }))
			}
		],
		dataset: [
			{
			seriesname: "Code Comprehension",
			data: chartData.map(item => ({ value: item.code_comprehen*10 }))
			},
			{
			seriesname: "Doc Comprehension",
			data: chartData.map(item => ({ value: item.doc_comprehen*10 }))
			},
			{
			seriesname: "Code Reusability",
			data: chartData.map(item => ({ value: item.code_reusability*10 }))
			},
			{
			seriesname: "Built Reusability",
			data: chartData.map(item => ({ value: item.built_reusability*10 }))
			},
			{
			seriesname: "Test Quality",
			data: chartData.map(item => ({ value: item.test_quality }))
			},
			{
				//adjust scale
			seriesname: "Team Activeness",
			data: chartData.map(item => ({ value: item.team_activeness/1000 }))
			}
		]
		};
		FusionCharts.ready(function() {
		const chart = new FusionCharts({
			type: "mscolumn2d",
			renderAt: "chart-container",
			width: "100%",
			height: "400",
			dataFormat: "json",
			dataSource
		});

		chart.render();
		});
		}


	function deleteProject(index){
		let tempIndex = 0;

		if(index == null){
			projects.slice(tempIndex, 1);
			chartRander(projects)
		}else{
			projects.splice(index, 1);
			chartRander(projects)
		}

		projects = projects;
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

			chartRander(projects)
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
							<!-- Fixed decimal digit to 2 -->
							<td>{(Math.round((project.code_comprehen*10)*100)/100).toFixed(2)}</td>
							<td>{(Math.round((project.doc_comprehen*10)*100)/100).toFixed(2)}</td>
							<td>{(Math.round((project.code_reusability*10)*100)/100).toFixed(2)}</td>
							<td>{(Math.round((project.built_reusability*10)*100)/100).toFixed(2)}</td>
							<td>{(Math.round((project.test_quality)*100)/100).toFixed(2)}</td>
							<td>{(Math.round((project.team_activeness/1000)*100)/100).toFixed(2)}</td>
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
  <div id="chart-container"></div>

  <Footer/>

  <link href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.0/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-9aIt2nRpC12Uk9gS9baDl411NQApFmC26EwAOH8WgZl5MYYxFfc+NcPb1dKGj7Sk" crossorigin="anonymous">