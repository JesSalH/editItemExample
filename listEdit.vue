<template>
   
    <div>
		
		<div class="title-bar">
            <div class="row">
                <div class="col-5">
                    <h1 class="page-title">Order Items </h1>
                </div>

                <div class="col-7 d-flex justify-content-end">
                    <div>
                        <!-- botones para mobile y para desktop -->
						<button id="createNewBtn" class="btn btn-primary btn-sm hide-on-mobile" v-on:click="addInlineClick"><i class="fas fa-plus"></i> Create New</button>
						<button id="createNewMobileBtn" class="btn btn-primary btn-sm hide-on-desktop" v-on:click="addInlineMobileClick"><i class="fas fa-plus"></i> Create New</button>
						<!--<button id="downloadExcelBtn" class="btn btn-secondary btn-sm" v-on:click="downloadExcel"><i class="far fa-file-excel"></i> Download Excel</button>-->
                    </div>
                </div>
            </div>
        </div>
		


        <div class="card content-panel">
            <div class="card-body">

				<v-server-table id="mainTable" :url="apiUrl" :columns="columns" :options="options" ref="entityTable">
					<!--<div slot="h__actions" slot-scope="props"><button id="createNewBtn" class="btn btn-primary btn-sm" v-on:click="addClick">Create New</button></div>-->
					<div slot="actions" slot-scope="props" class="actions-cell">
						<button id="updateBtn" class="btn btn-link btn-sm" @click="updateInlineClick(props.row)" v-if="entityId==props.row.id"><i class="fas fa-sync-alt"></i> Update</button>
						<button id="editBtn" class="btn btn-link btn-sm" @click="editInlineClick(props.row.id)" v-else><i class="far fa-edit"></i> Edit</button>
						<button id="cancelBtn" class="btn btn-link btn-sm" @click="cancelInlineClick()" v-if="entityId==props.row.id"><i class="fas fa-times"></i> Cancel</button>
						<button id="deleteBtn" class="btn btn-link text-danger btn-sm" @click="deleteClick(props.row.id)" v-else><i class="far fa-trash-alt"></i> Delete</button>
					</div>

                    <!-- Para mobile -->
					<div slot="mobile" slot-scope="props">
                        <p><b>Quantity</b> {{props.row.quantity}}</p>
                        <p><b>Purchase Price</b>{{  props.row.purchasePrice }} PP </p>

                        <button id="editMobileBtn" class="btn btn-primary btn-sm" @click="editInlineMobileClick(props.row.id)">Edit</button>
                        <button id="deleteMobileBtn" class="btn btn-danger btn-sm" @click="deleteClick(props.row.id)">Delete</button>
                    </div>

                    <!-- ojo a v-model y al props -->
                    <!-- columna quantity -->
                    <div slot="quantity" slot-scope="props">
                        <input id="quantityInput" type="number" name="quantity"
                               v-model="entity.quantity" v-validate="'required'" v-if="entityId==props.row.id" :class="{'form-control': true, 'is-invalid': errors.has('quantity') }"
                               data-vv-as="Quantity" />                               
                        <span v-else>{{props.row.quantity}}</span>
                    </div>
                    <!-- columna product id -->                   
                    <div slot="productId" slot-scope="props">
                            <!-- :custom-label, @search-change= -->
                        <multiselect v-model="productSearch.selectedItem" id="productIdInput" :custom-label="productSearch_label" track-by="id" placeholder="Type to search"
                                     :options="productSearch.list" :multiple="false" :searchable="true" :loading="productSearch.isLoading" :internal-search="false"
                                     :close-on-select="true" :show-no-results="false"
                                     :clear-on-select="true" @search-change="productSearch_find" @select="productSearch_select" v-if="entityId==props.row.id">
                            <template slot="clear" slot-scope="props">
                                <div class="multiselect__clear" v-if="productSearch.length" @mousedown.prevent.stop="clearAll(props.search)"></div>
                            </template><span slot="noResult">No elements found</span>
                        </multiselect>                       
                        <!-- product es una subtabla dentro de orderItem -->
                        <span v-else> {{ props.row.product.productName }}</span>
                    </div>

                    <!--hemos anyadido un watcher a entity.quantity que modifica el entity.purchasePrice -->
                    <div slot="purchasePrice" slot-scope="props">
                        <input id="purchasePriceInput" type="number" name="purchasePrice"
                               v-model="entity.purchasePrice" v-if="entityId==props.row.id" :class="{'form-control': true, 'is-invalid': errors.has('purchasePrice') }"
                               data-vv-as="Purchase Price" />
                                <!--ojo aparecen estas props pero lo que modificamos es el entity.purchasePrice -->
                        <span v-else> {{ props.row.purchasePrice }} </span>
                    </div>

                    <!-- create new Line -->
                    <dcsl-inline-grid-tr slot="prependBody" v-if="isAddingEntity">
                    <dcsl-inline-grid-td>
                        <!-- columna quantity -->
                        <input id="quantityInput" type="number" name="quantity"
                               v-model="entity.quantity" v-validate="'required'" :class="{'form-control': true, 'is-invalid': errors.has('quantity') }"
                               data-vv-as="Quantity" />
                    </dcsl-inline-grid-td>
                    <!-- col id -->
                    <dcsl-inline-grid-td>
                        <multiselect v-model="productSearch.selectedItem" id="productIdInput" :custom-label="productSearch_label" track-by="id" placeholder="Type to search"
                                     :options="productSearch.list" :multiple="false" :searchable="true" :loading="productSearch.isLoading" :internal-search="false"
                                     :close-on-select="true" :show-no-results="false"
                                     :clear-on-select="true" @search-change="productSearch_find" @select="productSearch_select">
                            <template slot="clear" slot-scope="props">
                                <div class="multiselect__clear" v-if="productSearch.length" @mousedown.prevent.stop="clearAll(props.search)"></div>
                            </template><span slot="noResult">No elements found</span>
                        </multiselect>
                    </dcsl-inline-grid-td>
                    <dcsl-inline-grid-td>                   
                        <!-- col price -->
                        <input id="purchasePriceInput" type="number" name="purchasePrice"
                               v-model="entity.purchasePrice" :class="{'form-control': true, 'is-invalid': errors.has('purchasePrice') }"
                               data-vv-as="Purchase Price" />
                    </dcsl-inline-grid-td>

						<dcsl-inline-grid-td>
							<div class="text-nowrap">
								<button id="createBtn" class="btn btn-primary btn-sm" @click="saveNewInline()">Save</button>
								<button id="cancelBtn" class="btn btn-secondary btn-sm" @click="cancelAddInline()">Cancel</button>
							</div>
						</dcsl-inline-grid-td>
					</dcsl-inline-grid-tr>
				</v-server-table>

		    </div>
        </div>


        <!--add/edit modal-->
        <v-modalPopup :visible="showEntity" :title="'Order Items Add/Edit'" @close="closeEntity">
            <template slot="content">

                <div class="form-group row">
                    <label for="quantityInput" class="col-sm-3 col-form-label">Quantity</label>
                    <div class="col-sm-9">
                        <input id="quantityInput" type="number" name="quantity"
                               v-model="entity.quantity" v-validate="'required'" :class="{'form-control': true, 'is-invalid': errors.has('quantity') }"
                               data-vv-as="Quantity" />
                        <div class="invalid-feedback" v-show="errors.has('quantity')">{{ errors.first('quantity') }}</div>
                    </div>
                </div>
                <div class="form-group row">
                    <label for="productIdInput" class="col-sm-3 col-form-label">Product</label>
                    <div class="col-sm-9">
                        <multiselect v-model="productSearch.selectedItem" id="productIdInput" :custom-label="productSearch_label" track-by="id" placeholder="Type to search"
                                     :options="productSearch.list" :multiple="false" :searchable="true" :loading="productSearch.isLoading" :internal-search="false"
                                     :close-on-select="true" :show-no-results="false"
                                     :clear-on-select="true" @search-change="productSearch_find" @select="productSearch_select">
                            <template slot="clear" slot-scope="props">
                                <div class="multiselect__clear" v-if="productSearch.length" @mousedown.prevent.stop="clearAll(props.search)"></div>
                            </template><span slot="noResult">No elements found</span>
                        </multiselect>
                        <div class="invalid-feedback" v-show="errors.has('productId')">{{ errors.first('productId') }}</div>
                    </div>
                </div>
                <div class="form-group row">
                    <label for="purchasePriceInput" class="col-sm-3 col-form-label">Purchase Price</label>
                    <div class="col-sm-9">
                        <input id="purchasePriceInput" type="number" name="purchasePrice"
                               v-model="entity.purchasePrice" 
                               :class="{'form-control': true, 'is-invalid': errors.has('purchasePrice') }"
                               data-vv-as="Purchase Price"
                        />
                        <div class="invalid-feedback" v-show="errors.has('purchasePrice')">{{ errors.first('purchasePrice') }}</div>
                    </div>
                </div>

            </template>

            <template slot="footer">
                <!--<button type="button" id="confirmSaveBtn" class="btn btn-primary" :disabled="errors.any()" v-on:click="saveInlineMobileEntity">Save changes</button>-->
                <button type="button" id="closePopupBtn" class="btn btn-secondary" v-on:click="closeEntity">Close</button>
                <button type="button" id="confirmSaveBtn" class="btn btn-primary" :disabled="errors.any()" v-on:click="saveEntity">Save changes</button>
            </template>
        </v-modalPopup>


        <!--delete modal-->
        <v-modalPopup :visible="showDeleteEntity" :title="'Delete Item'" @close="cancelDelete"
                      :message="'Are you sure you want to delete this item?'">

            <template slot="footer">
                <button type="button" id="confirmDeleteBtn" class="btn btn-danger" v-on:click="deleteEntity">Delete</button>
                <button type="button" id="cancelDeleteBtn" class="btn btn-secondary" v-on:click="cancelDelete">Cancel</button>
            </template>

        </v-modalPopup>

        <!--alert modal-->
        <v-modalPopup :bus="this" :namespace="'alertModal'" :visible="alertVisible" @close="hideAlert"></v-modalPopup>

    </div>
</template>


<script lang="ts">
    import Vue from "vue";
    import Component from "vue-class-component";
    import * as _ from "lodash";
    import { Mixins } from "vue-mixin-decorator";
    // Watch
    import { Prop, Watch } from "vue-property-decorator";
    import { DatePicker } from "element-ui";
    import DcslInlineGridEditTr from "../../components/dcslcomponents/dcsl-inline-grid-tr.vue";
    import DcslInlineGridEditTd from "../../components/dcslcomponents/dcsl-inline-grid-td.vue";
    import BaseCrudMixin from "../../js/baseCrudMixin";
    import OrderItemModel from "../../js/dataModels/orderItemModel";
    import Utils from "../util/utilsTs";

    @Component({
        name: "orderItemsListEdit",
        components: {
            "datepicker": DatePicker,
            "dcsl-inline-grid-tr": DcslInlineGridEditTr,
            "dcsl-inline-grid-td": DcslInlineGridEditTd
        },
    })
    export default class OrderItems extends Mixins<BaseCrudMixin>(BaseCrudMixin) {

        @Watch('entity.quantity')
        onChildChanged(val: number, oldVal: number) {
            this.updatePurchasePrice(val, oldVal);
        }

        //entity
        entity: OrderItemModel;
        isAddingEntity: boolean;
        apiUrl: string;
        downloadUrl: string;
        fileUrl: string;
        showEntity: boolean;
        columns: string[];
        options: any;
        entityId: number;
        showDeleteEntity: boolean;
        alertVisible: boolean;
        productSearch: any;
        //orderId -> prop con la que se pasa la id del padre a este componente para que esten enlazados
        @Prop(Number) orderId!: number;

        constructor() {
            super();
            this.entity = new OrderItemModel();
            this.isAddingEntity = false;
            this.apiUrl = Vue.prototype.$rootApiPath + "OrderItem/";
            this.downloadUrl = this.apiUrl + "DownloadFile/";
            this.fileUrl = this.apiUrl + "UploadFile/";
            this.showEntity = false;
            this.columns = [
                    "mobile",
                    "quantity",
                    "productId",
                    "purchasePrice",
                    "actions"
            ];
            this.options = {
                headings: Utils.getColumnHeadings(this.columns),
                filterByColumn: true,
                filterable: Utils.getCols(this.columns, ["productId", "product"]),
                listColumns: {
                },
                sortable: Utils.getCols(this.columns, ["productId", "product"]),
				columnsClasses: Utils.getColumnsClasses(this.columns),
                params: {
                    orderId: ""
                }
            };
            this.entityId = 0;
            this.showDeleteEntity = false;
            this.alertVisible = false;
            this.productSearch = {
                isLoading: false,
                list: [],
                selectedItem: null
            };
        }
        
        
        created(): void {
            this.initLookups(["Order","Product"]);
            this.options.params.orderId = this.orderId;

            this.lookupFetchers["Order"]
            //  .Fields(["id", "name"])
            //  .Api("Order/")
                .AfterFetch((records) => {
                    this.options.listColumns.orderId = records.map((r) => {
                        return { id: r.id, text: r.name };
                    });
                });
            this.lookupFetchers["Product"]
            //  .Fields(["id", "name"])
            //  .Api("Product/")
                .AfterFetch((records) => {
                    this.options.listColumns.productId = records.map((r) => {
                        return { id: r.id, text: r.name };
                    });
                });

            this.fetchLookups();
            this.entityInitCallbacks.push(this.productSearch_callback);
            //primero llamada en el create
            this.productSearch_find("", () => this.productSearch_callback());

        };

        //actualizamos purchase price cuando se carga la pagina
        mounted(): void {
        this.$nextTick(function () {        
            console.log("page mounted");
            
            console.log('product: ' + this.entity.product.productName);
            console.log('quantity: ' + this.entity.quantity);
            console.log('product price: ' + this.entity.product.price);
            //just redefine this vale
            this.entity.purchasePrice = this.entity.quantity * this.entity.product.price;
            console.log('new purchase price: ' + this.entity.purchasePrice);
        })
    }

        addInlineClick(): void {   //addClick: function () {
            this.initEntity();
            this.isAddingEntity = true;
        };
        editInlineClick(id: number): void {    //editClick: function (id) {
            this.$http.get(this.apiUrl + id).then(this.inlineRead, this.dataReadFailure);
            this.entityId = id;
        };
        inlineRead(event: any): void {
            this.$mapper.from(event.body).to(this.entity).map();
            this.entityInitCallbacks.forEach(cb => cb());
        };
        cancelAddInline(): void {
            this.initEntity();
            this.isAddingEntity = false;
        };
        saveNewInline(): void {
            this.$http.post(this.apiUrl, this.entity).then(this.finishedInlineCreate, this.dataReadFailure);
        };
        finishedInlineCreate(): void {
            (this as any).$refs.entityTable.refresh();
            this.initEntity();
            this.isAddingEntity = false;
        };
        cancelInlineClick(): void {
            this.entityId = 0;
            (this as any).$refs.entityTable.refresh();
        };
        updateInlineClick(row: any): void {
            //Validation needed
            this.$http.put(this.apiUrl + row.id, this.entity).then(this.entityId = 0, this.dataReadFailure)
                .then((this.$refs.entityTable as any).refresh);
        };
        editInlineMobileClick(id: number): void {
            this.$http.get(this.apiUrl + id).then(this.dataRead, this.dataReadFailure);
        };
        addInlineMobileClick(): void {
            this.initEntity();
            this.showEntity = true;
        };
        findLookupValue(lookupName, id, propName): string {
            if (!propName) propName = "name";
            const lookupObj = this.lookups[lookupName].find(x => x.id == id);
            return (lookupObj) ? lookupObj[propName] : "";
        }

        //Overridden methods
        getSaveCallback(): any {
            return this.updatedEntity;
        };
        initEntity(): void {
            this.entity = new OrderItemModel();
            this.entity.orderId = this.orderId;
            this.entityInitCallbacks.forEach(cb => cb());
        };

        productSearch_find(query: any, callback?: any): void {
            this.productSearch.isLoading = true;
            var searchUrl = Vue.prototype.$rootApiPath + "Product/?query=" + query;
            this.$http.get(searchUrl).then((event: any): void => {
                this.productSearch.list = event.body.data;
                this.productSearch.isLoading = false;
                if (callback) callback();
            });
        }

        //cambiar de producto
        productSearch_select(item: any): void {
            this.entity.productId = item.id;
            this.entity.product = item;
            //1.- tb reseteamos la quantity de esa fila a 1
            this.entity.quantity = 1;
            // 2.- actualizamos precio con quantity 1         
            this.updatePurchasePrice(1, 0);
        }

        productSearch_callback(): void {
            if (this.entity.id > 0) {
                this.productSearch.selectedItem = this.productSearch.list.find(item => item.id == this.entity.productId);
            } else {
                this.productSearch.selectedItem = null;
            }
        }

        productSearch_label(item: any): string {
            return `${item.productName}`;
        }


        //funcion que actuliza el precio de esa fila
        // nuevo valor por el precio de ese producto (entramos en la subtabla product)
        updatePurchasePrice(val, oldVal): void {

            console.log('Watcher called');
            console.log('val: ' + val);
            console.log('product: ' + this.entity.product.productName);
            console.log('quantity: ' + this.entity.quantity);
            console.log('product price: ' + this.entity.product.price);
            //just redefine this value
            this.entity.purchasePrice = val * this.entity.product.price;
        }
    }
</script>