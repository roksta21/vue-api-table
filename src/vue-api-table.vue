<template>
	<div>
		<div class="row" v-if="loaded">
			<div class="col-md-6"></div>
			<div class="col-md-12">
				<select class="form-control pull-right" style="width: 40%" v-model="per_page" @change="fetch">
					<option value="20">Show 20</option>
					<option value="50">Show 50</option>
					<option value="100">Show 100</option>
				</select>
			</div>
			<div class="col-md-12">
				<table class="table table-striped table-hover table-responsive">
					<thead>
						<tr>
							<th v-for="column in the_columns">{{ column }}</th>
						</tr>
					</thead>
					<tbody>
						<tr v-for="row in data" class="link" @click="goTo(row)">
							<td v-for="i in the_columns">{{ row[columns[i]] }}</td>
						</tr>
						<tr v-if="data.length < 1">
							<td :colspan="columns_count">
								<div class="alert alert-warning">No data</div>
							</td>
						</tr>
					</tbody>
				</table>
				<ul class="pagination">
					<template v-if="total > (page * per_page)">
						<li v-if="this.page > 1"><a href="#" @click.prevent="changePage(page-1)">Previous</a></li>
						<li><a href="#">{{ this.page }}</a></li>
						<li v-if="this.page > this.pages_count"><a href="#" @click.prevent="changePage">Next</a></li>
						<li v-if="this.page < this.pages_count"><a href="#" @click.prevent="changePage('next')">Next</a></li>
					</template>
				</ul>
			</div>
		</div>
		<div v-if="!loaded" style="text-align: center">
			<img src="/public/images/loading.gif">
		</div>
	</div>
</template>

<script>
	export default {
		mounted() {
			this.fetch();
		},

		data() {
			return {
				data: [],
				page: 1,
				per_page: 20,
				total: 0,
				loaded: false
			}
		},

		props: {
			url: {
				type: String,
				default() {
					return '';
				}
			},

			classes: {
				type: String,
				default() {
					return 'table table-striped table-hover table-responsive';
				}
			},

			columns: {
				type: Object,
				default() {
					return {};
				}
			}
		},

		methods: {
			fetch() {
				let url = this.url + '?show=' + this.per_page +'&page=' + this.page;
				this.loaded = false;
				this.data = [];

				axios.get(url).then(response => {
					response.data.results.forEach(set => {
						let mapped = [];

						this.the_mappings.forEach(column => {
							let nestings = column.split('.');

							if (nestings.length == 1) {
								mapped[column] = set[nestings[0]];
							}

							if (nestings.length == 2) { 
								mapped[column] = set[nestings[0]][nestings[1]];
							}

							if (nestings.length == 3) {
								mapped[column] = set[nestings[0]][nestings[1]][nestings[2]];
							}
							
						});

						this.data.push(mapped);
					});

					this.page = response.data.page;
					this.per_page = response.data.per_page;
					this.total = response.data.total;
					this.loaded = true;
				});
			},

			changePage(direction) {
				if (direction == 'next') {
					this.page = parseInt(this.page) + 1;
				} else {
					this.page = parseInt(this.page) - 1;
				}
				
				this.fetch();
			}
		},

		computed: {
			the_columns() {
				return Object.keys(this.columns);
			},

			the_mappings() {
				return Object.values(this.columns);
			},

			columns_count() {
				return this.the_columns.length;
			},

			pages_count() {
				return Math.ceil(this.total / this.per_page);
			}
		}
	}
</script>