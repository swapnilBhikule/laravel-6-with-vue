<template>
	<div class="modal fade" id="add-product" tabindex="-1" role="dialog" aria-labelledby="add-product" aria-hidden="true">
		<div class="modal-dialog" role="document">
			<div class="modal-content">
				<div class="modal-header">
					<h5 class="modal-title" id="add-product-title">Add Product</h5>
					<button type="button" class="close" data-dismiss="modal" aria-label="Close">
						<span aria-hidden="true">&times;</span>
					</button>
				</div><!-- end of div modal-header -->
				<div class="modal-body">
					<form name="add-product-form" method="post" @submit.prevent="validateBeforeSubmit">
						<div class="form-group">
							<label for="name">Name</label>
							<input type="text" id="name" name="name" class="form-control" v-model="name"
							   required maxlength="60">
						</div><!-- end of div form-group -->

						<div class="form-group">
							<label for="description">Description</label>
							<textarea id="description" name="description" class="form-control" v-model="description"
									  required></textarea>
						</div><!-- end of div form-group -->

						<div class="form-group">
							<label for="category">Category</label>
							<select id="category" name="category" class="form-control" v-model="category_id" required>
								<option value="-1" disabled>Select Category</option>
								<option v-for="category in categories" :value="category.id">{{ category.title }}</option>
							</select>
						</div><!-- end of div form-group -->

						<div class="form-group">
							<label for="price">Price</label>
							<input type="text" id="price" name="price" class="form-control" v-model="price"
								   required maxlength="8">
						</div><!-- end of div form-group -->

						<div class="form-group">
							<label for="image">Image</label>
							<input type="file" id="image" name="image" class="form-control" @change="onFileChange" required>
						</div><!-- end of div form-group -->

						<div class="form-group text-right">
							<button type="submit" class="btn btn-success" title="Add Product" :disabled="ajax_loading">
								<span v-if="ajax_loading">Adding...</span>
								<span v-else>Add Product</span>
							</button>
						</div><!-- end of div form-group -->
					</form><!-- end of form add-product-form -->
				</div><!-- end of div modal-body -->
			</div><!-- end of div modal-content -->
		</div><!-- end of div modal-dialog -->
	</div><!-- end of div modal -->
</template>

<script>
	export default {
		props: ['categories'],
		data() {
			return {
				name: '',
				category_id: '-1',
				category: '',
				description: '',
				image: '',
				price: '',
				ajax_loading: false
			}
		},
		methods: {
			onFileChange(e) {
				let files = e.target.files || e.dataTransfer.files;
				if (! files.length) {
					return;
				}

				this.createImage(files[0]);
			},
			createImage(file) {
				let reader = new FileReader();

				reader.onload = (e) => {
					this.image = e.target.result;
				};
				reader.readAsDataURL(file);
			},
			validateBeforeSubmit() {
				let category = _.find(this.categories, {id: this.category_id});

				if (typeof category === 'undefined') {
					izitoast.error({
						title: 'Sorry!',
						message: 'Please select the proper category',
						position: 'topCenter'
					});
				}

				this.category = category.title;

				this.submit();
			},
			submit() {
				this.ajax_loading = true;
				this.$store.dispatch('products/addProduct', {
					name: this.name,
					category_id: this.category_id,
					category: this.category,
					description: this.description,
					image: this.image,
					price: this.price
				})
					.then(response => {
						this.name = '';
						this.category_id = '-1';
						this.category = '';
						this.description = '';
						this.price  = '';
						this.image = '';
						this.ajax_loading = false;
						$("#add-product").modal('hide');

						if (response.status !== 200 && response.error) {
							this.displaySuccessError(response);
						}
					})
					.catch(error => {
						this.ajax_loading = false;

						this.displayErrors(error);
					});
			}
		}
	}
</script>

<style scoped>

</style>
