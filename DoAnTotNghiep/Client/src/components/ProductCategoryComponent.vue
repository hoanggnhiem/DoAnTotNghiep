<template>
    <div>
        <ProductCategoryToolbarComponent  
            v-on:showDialogProduct = "showDialogProduct" 
            v-on:setIsProductCategory ="setIsProductCategory" 
            v-on:showDialogProductCategory = "showDialogProductCategory"
            v-on:btnDeleteRecord = "btnDeleteRecord"
        />

        <ProductCategoryGridComponent 
            v-bind:productCategories="productCategories" 
            v-on:passIsAddOrEdit = "passIsAddOrEdit"
            v-on:passProductCategoryEdit = "passProductCategoryEdit"
            v-on:showDialogProductCategory = "showDialogProductCategory"
            v-on:insertProductCategoryIdToListId = "insertProductCategoryIdToListId"
        />

        <ProductCategoryPopupComponent 
            v-if="showDialog" 
            v-on:confirmHidedDialog = "confirmHidedDialog" 
            v-bind:listFieldNotValidRequired = "ListFieldNotValidRequired"
            v-bind:isAddOrEdit = "isAddOrEdit"
            v-on:updateProductCategory = "updateProductCategory"
            v-on:showToastSuccess = "showToastSuccess"
            v-on:hidedDialogProductCategory = "hidedDialogProductCategory"
            v-bind:productCategoryEdit = "productCategoryEdit"
        />


        <PopupConfirmComponent 
            v-show = "isShowDialogConfirm"
            v-bind:messageConfirm = "messageConfirm"
            v-bind:isCancelSaveConfirm = "isCancelSaveConfirm"
            v-on:btnHideDialogConfirm = "btnHideDialogConfirm"
            v-bind:isDeleteConfirm = "isDeleteConfirm"
            v-on:btnNotHideDialogConfirm = "btnNotHideDialogConfirm"
            v-on:btnConfirmDelete = "btnConfirmDelete"
          
        />

        <ToastSuccessComponent v-show = "isShowToastSuccess" v-bind:messageSuccess = "messageSuccess" />

        <PopupValidateComponent 
            v-show="isShowDialogValidate" 
            v-bind:messageNotValid = "messageNotValid" 
            v-on:btnHideDialogNotValid = "btnHideDialogNotValid"
        />


    </div>
</template>

<script>


import axios from 'axios'
import PopupValidateComponent from './Popup/PopupValidateComponent.vue'

/* JS */

import {Resource} from '../../src/assets/js/common/resource'
import {Enum} from '../../src/assets/js/common/Enum'
import {ToastJS} from '../../src/assets/js/common/toastJs'

/* import component */
import ProductCategoryToolbarComponent from '../components/Toolbar/ProductCategoryToolbarComponent.vue'
import ProductCategoryGridComponent from '../components/Grid/ProductCategoryGridComponent.vue'

import ProductCategoryPopupComponent from '../components/Popup/ProductCategoryPopupComponent.vue'

import ToastSuccessComponent from '../components/Toast/ToastSuccessComponent.vue'
import PopupConfirmComponent from '../components/Popup/PopupConfirmComponent.vue'

export default {
    name : "product-category-component",
    components : {
       ProductCategoryToolbarComponent,
       ProductCategoryGridComponent,
       ProductCategoryPopupComponent,
       ToastSuccessComponent,
       PopupConfirmComponent,
       PopupValidateComponent
    },
    data(){
        return{
            productCategories  : [],
            isShowLoader : false,
            currentPage : 1,
            showDialog : false,
            isAddOrEdit : Enum.FormModel.View,
            isShowDialogConfirm : false,

            /**
             * danh s??ch c??c tr?????ng ch??a valid b???t bu???c nh???p khi l??u d??? li???u
             */
            ListFieldNotValidRequired : [],
            isShowToastSuccess : false,
            messageSuccess : "",

            /**
             * D??? li???u n??y d??ng ????? load l??n form chi ti???t
             */
            productCategoryEdit : Object.assign({}),

            isCancelSaveConfirm : false,
            isDeleteConfirm : false,
            listProductCategoryIdToDelete : [],
            isShowDialogValidate : false,
            messageNotValid :"",
        }
    },

    methods : {

        /**
         * Method d??ng ????? x??a s???n ph???m khi ng?????i d??ng x??c nh???n x??a
         * CreatedBy : NXHoang 
        */
        btnConfirmDelete : function()
        {
            try {
                let me = this;
                let listId = '';
                this.listProductCategoryIdToDelete.forEach(item => {
                    listId += item + ',';
                });
                listId = listId.slice(0,listId.length - 1);

                
                //g???i API x??a 1 b???n ghi c???a s???n ph???m
                
                axios.delete(Resource.API.Host + Resource.API.ProductCategory + `/deleteMulti?listId=${listId}`)
                    .then(() => {
                        me.isShowDialogConfirm = false;
                        //???n b??? c??c button c???a dialog confirm
                        me.isCancelSaveConfirm = false;
                        me.isDeleteConfirm = false;
                        me.showToastSuccess(ToastJS.deleteMessageSuccess);
                        this.listProductCategoryIdToDelete = [];
                        me.getData();
                    })
                    .catch(() => {
                        me.isShowDialogConfirm = false;
                        //???n b??? c??c button c???a dialog confirm
                        me.isCancelSaveConfirm = false;
                        me.isDeleteConfirm = false;
                    })
            } catch (error) {
                console.log(error)
            }
        },


        /**
         * X??? l?? khi nh???n button H???y tr??n dialog th??ng b??o not valid c??c tr?????ng b???t bu???c nh???p
         * CreatedBy : NXHoang (22/11/2021)
         */
        btnHideDialogNotValid : function()
        {
            try {
                // this.ListFieldNotValidRequired = [];
                this.messageNotValid = "";
                this.isShowDialogValidate = false;
                
            } catch (error) {
                console.log(error)
            }
        },

        /** 
         * S??? ki???n d??ng ????? x??a c??c b???n ghi ???????c ch???n 
         * CreatedBy : NXHoang (19/11/2021)
        */
        btnDeleteRecord : function()
        {
            try {
                if(this.listProductCategoryIdToDelete.length == 0)
                {
                    this.showDialogNotValid(ToastJS.deleteMessageWarning)
                }
                else{

                    this.isShowDialogConfirm = true;
                    this.isDeleteConfirm = true;
                    this.messageConfirm = Resource.VN.MessageDialogConfirm.deleteMessage;
                    
                }
            } catch (error) {
                console.log(error)
            }
        },

        /**
         * Method show dialog not valid
        */
        showDialogNotValid : function(message)
        {
            this.messageNotValid = '<span class = "text-notvalid-dialog">' +  message  + '</span>';
            this.isShowDialogValidate = true;
        },

        insertProductCategoryIdToListId : function(listIdSelectedRecord)
        {
            this.listProductCategoryIdToDelete = listIdSelectedRecord;
        },


        /**
         * Method x??c nh???n vi???c h???y ????ng popup th??m/s???a s???n ph???m
         * CreatedBy : NXHoang 
        */
        btnNotHideDialogConfirm : function()
        {
            try {
                this.isShowDialogConfirm = false;
                this.isCancelSaveConfirm = false;
                this.isDeleteConfirm = false;
            } catch (error) {
                console.log(error);
            }
        },

        /**
         * Method d??ng ????? ????ng popup th??m/s???a s???n ph???m
         * CreatedBy : NXHoang
        */
        btnHideDialogConfirm : function(){
            try {
                this.isShowDialogConfirm = false;
                this.showDialog = false;
                //???n b??? c??c button c???a dialog confirm
                this.isCancelSaveConfirm = false;
                this.isDeleteConfirm = false;
            } catch (error) {
                console.log(error);
            }
        },


        passProductCategoryEdit : function(productCategoryEdit)
        {   
            this.productCategoryEdit = Object.assign({},productCategoryEdit);
        },

        /**
         * Method ????? check xem ??ang th??m hay ??ang s???a
         * CreatedBy : NXHoang
        */
        passIsAddOrEdit : function(value)
        {
            this.isAddOrEdit = value;
        },

        /*      
        * method hi???n th??? dialog confirm cho ng?????i d??ng v??? vi???c ????ng popup th??m/s???a s???n ph???m
        * CreatedBy : NXHoang
        */
        confirmHidedDialog : function(message , isCancelSaveConfirm){
            try {
                this.isShowDialogConfirm = true;
                this.messageConfirm = message;
                this.isCancelSaveConfirm = isCancelSaveConfirm;
            } catch (error) {
                console.log(error);
            }
        },


        /**
         * L???y danh s??ch danh m???c s???n ph???m
         * CreatedBy : NXHoang
        */
        getData(){
            var me = this;
            this.isShowLoader = true;
            // axios.get(Resource.API.Host + Resource.API.GetProductAp)i
            axios.get(Resource.API.Host + Resource.API.ProductCategory + Resource.API.searchProductCategory(Resource.SearchPaging.textSearch, Resource.SearchPaging.PageSize ,  me.currentPage))
            .then(res => {
                this.isShowLoader = false;
                me.productCategories = res.data.data;
                // me.totalPage = res.data.totalPage;
                // me.totalRecord = res.data.totalRecord;
            })
            .catch(error => {
                // this.isShowLoader = false;
                console.log(error)
            })
        },

        /**
         * Method th???c hi???n show dialog chi ti???t s???n ph???m
         * createdBy : NXHoang 
        */
        showDialogProductCategory(){
            this.showDialog = true;
        },

        /**
         * Method d??ng ????? ????nh d???u l?? th??m danh m???c s???n ph???m
         * CreatedBy : NXHoang 
         */
        setIsProductCategory()
        {
            this.isAddOrEdit = Enum.FormModel.Add;
        },

        /**
         * Method ???n dialog nh???p li???u
         * CreatedBy : NXHoang 31/07/2022
        */
        hidedDialogProducCategort : function()
        {
            this.showDialog = false;
        },

        /**
         * Method d??ng ????? show toast th??ng b??o Th??m/S???a th??nh c??ng nh??n vi??n
         * CreatedBy : NXHoang
         */
        showToastSuccess : function(message){
            try {
                this.messageSuccess = message;
                this.isShowToastSuccess = true;
                setTimeout(() => {
                    this.isShowToastSuccess = false
                } , 3000)
            } catch (error) {
                console.log(error);
            }
        },

        /**
         * Method ???n dialog nh???p li???u
         * CreatedBy : NXHoang 31/07/2022
        */
        hidedDialogProductCategory : function()
        {
            this.showDialog = false;
        },


        /**
         * Method Load d??? li???u grid
         * CreatedBy : NXHoang 31/07/2022
        */
        updateProductCategory : function()
        {
            this.getData();
        },  

    },

    created() {
        this.getData();
    },
}
</script>

<style>

</style>