<template>  

<div>
    
    <ProductToolbarComponent 
        v-on:showDialogProduct = "showDialogProduct"
        v-on:setIsProduct = "setIsProduct"
        v-on:btnDeleteRecord = "btnDeleteRecord"
    />
    <ProductGridComponent 
        v-bind:products ="products" 
        v-on:showDialogProduct = "showDialogProduct"
        v-on:passProductEdit = "passProductEdit"
        v-on:passIsAddOrEdit = "passIsAddOrEdit"
        v-bind:isShowLoader = "isShowLoader"
        v-on:insertProductIdToListId = "insertProductIdToListId"
        v-on:btnPaging = "btnPaging"
        v-bind:totalPage = "totalPage"
        v-bind:totalRecord = "totalRecord"
    />
    
    <ProductPopupComponent 
        v-if="showDialog"
        v-bind:productCategory = "productCategory" 
        v-on:itemComboboxClick = "itemComboboxClick"
        v-on:confirmHidedDialog = "confirmHidedDialog"
        v-bind:isAddOrEdit = "isAddOrEdit"
        v-bind:listFieldNotValidRequired = "ListFieldNotValidRequired"
        v-on:hidedDialogProduct = "hidedDialogProduct"
        v-on:showToastSuccess = "showToastSuccess"
        v-bind:productEdit = "productEdit"
        v-on:updateProduct = "updateProduct"
       
    />


    <PopupConfirmComponent 
        v-show = "isShowDialogConfirm"
        v-bind:messageConfirm = "messageConfirm"
        v-bind:isCancelSaveConfirm = "isCancelSaveConfirm"
        v-bind:isDeleteConfirm = "isDeleteConfirm"
        v-on:btnHideDialogConfirm = "btnHideDialogConfirm"
        v-on:btnNotHideDialogConfirm = "btnNotHideDialogConfirm"
        v-on:updateProduct = "updateProduct"
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


/* Import Component */

import ProductGridComponent from './Grid/ProductGridComponent.vue'
import ProductToolbarComponent from './Toolbar/ProductToolbarComponent.vue'
import ProductPopupComponent from './Popup/ProductPopupComponent.vue'
import PopupConfirmComponent from './Popup/PopupConfirmComponent.vue'

import ToastSuccessComponent from './Toast/ToastSuccessComponent.vue'
import PopupValidateComponent from './Popup/PopupValidateComponent.vue'



/* Import library */
import axios from 'axios'

/* File Javascript */
import {Resource} from '../assets/js/common/resource'
import {CommonFunction} from '../assets/js/common/commonFunction'
import {Enum} from '../assets/js/common/Enum'
import {ToastJS} from '../assets/js/common/toastJs'

export default {
    name : "product-compoment",
    components : {
        ProductGridComponent,
        ProductToolbarComponent,
        ProductPopupComponent,
        PopupConfirmComponent,
        ToastSuccessComponent,
        PopupValidateComponent
    },
    data(){
        return{
            products : [],
            productCategory : [],
            isShowDialogConfirm : false,
            showDialog : false,
            isCancelSaveConfirm : false,
            isDeleteConfirm : false,
            isAddOrEdit : Enum.FormModel.View,
            /**
             * danh s??ch c??c tr?????ng ch??a valid b???t bu???c nh???p khi l??u d??? li???u
             */
            ListFieldNotValidRequired : [],
            isShowToastSuccess : false,

             /**
             * D??? li???u n??y d??ng ????? load l??n form chi ti???t
             */
            productEdit : Object.assign({}),

            isShowLoader : false,

            listProductIdToDelete : [],
            isShowDialogValidate : false,
            messageNotValid :"",

            totalPage : 1,
            totalRecord : 0,
            currentPage : 1,
        }
    },

    methods : {


        /**
         * S??? ki???n k??ch ho???t khi chuy???n sang trang kh??c
         * createdBy : NXHoang 
        */
        btnPaging : function(currentPage)
        {
            var self = this;
            self.currentPage = currentPage;
            this.isShowLoader = true;
            axios.get(Resource.API.Host + Resource.API.Product + Resource.API.searchProduct(Resource.SearchPaging.textSearch, Resource.SearchPaging.PageSize ,  currentPage))
            .then(res => {
                self.products = res.data.data;
                self.totalPage = res.data.totalPage;
                self.totalRecord = res.data.totalRecord;
                this.isShowLoader = false;
            })
            .catch((error) => {
                console.log(error)
                this.isShowLoader = false;
            })
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
         * Method d??ng ????? x??a s???n ph???m khi ng?????i d??ng x??c nh???n x??a
         * CreatedBy : NXHoang 
        */
        btnConfirmDelete : function()
        {
            try {
                let me = this;
                let listId = '';
                this.listProductIdToDelete.forEach(item => {
                    listId += item + ',';
                });
                listId = listId.slice(0,listId.length - 1);

                
                //g???i API x??a 1 b???n ghi c???a s???n ph???m
                
                axios.delete(Resource.API.Host + Resource.API.Product + `/deleteMulti?listId=${listId}`)
                    .then(() => {
                        me.isShowDialogConfirm = false;
                        //???n b??? c??c button c???a dialog confirm
                        me.isCancelSaveConfirm = false;
                        me.isDeleteConfirm = false;
                        me.showToastSuccess(ToastJS.deleteMessageSuccess);
                        this.listProductIdToDelete = [];
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

        insertProductIdToListId : function(listIdSelectedRecord)
        {
            console.log(listIdSelectedRecord);
            this.listProductIdToDelete = listIdSelectedRecord;
        },

        /**
         * S??? ki???n d??ng ????? x??a c??c b???n ghi ???????c ch???n 
         * CreatedBy : NXHoang (19/11/2021)
        */
        btnDeleteRecord : function()
        {
            try {
                if(this.listProductIdToDelete.length == 0)
                {
                    this.showDialogNotValid(ToastJS.deleteMessageWarning)
                    // this.showToastWarning(ToastJS.deleteMessageWarning);
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

        passProductEdit : function(productEdit)
        {   
            this.productEdit = Object.assign({},productEdit);
        },

        // /**
        //  * Method show popup th???c hi???n vi???c s???a s???n ph???m
        //  * CreatedBy : NXHoang
        // */
        // showDialogProduct : function()
        // {
        //     this.showDialog = true;
        // },
        /**
         * Method ????? check xem ??ang th??m hay ??ang s???a
         * CreatedBy : NXHoang
        */
        passIsAddOrEdit : function(value)
        {
            this.isAddOrEdit = value;
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
         * Method Load d??? li???u grid
         * CreatedBy : NXHoang 31/07/2022
        */
        updateProduct : function()
        {
            this.getData();
        },  

        /**
         * Method ???n dialog nh???p li???u
         * CreatedBy : NXHoang 31/07/2022
        */
        hidedDialogProduct : function()
        {
            this.showDialog = false;
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
         * Method th???c hi???n vi???c selected c??c item c???a combobox
         * CreatedBy : NXHoang 
         */
        itemComboboxClick(data){
      
            CommonFunction.selectedItemCombobox(data);
        },

        /**
         * L???y danh s??ch s???n ph???m
         * CreatedBy : NXHoang (30/07/2022)
        */
        getData(){
            var me = this;
            this.isShowLoader = true;
            // axios.get(Resource.API.Host + Resource.API.GetProductApi)
            axios.get(Resource.API.Host + Resource.API.Product + Resource.API.searchProduct(Resource.SearchPaging.textSearch, Resource.SearchPaging.PageSize ,  me.currentPage))
            .then(res => {
                this.isShowLoader = false;
                me.products = res.data.data;
                me.totalPage = res.data.totalPage;
                me.totalRecord = res.data.totalRecord;
                this.isShowLoader = false;
            })
            .catch(error => {
                this.isShowLoader = false;
                console.log(error)
            })
        },

        /**
         * L???y danh s??ch danh m???c s???n ph???m
         * CreatedBy : NXHoang (30/07/2022)
        */
        getProductCategories(){
            var me = this;
            axios.get(Resource.API.Host + Resource.API.GetProductCategory)
            .then(res => {
                me.productCategory = res.data.result;
            })
            .catch(error => {
                console.log(error)
            })
        },


        /**
         * Method th???c hi???n show dialog chi ti???t s???n ph???m
         * createdBy : NXHoang 
        */
        showDialogProduct(){
             
            this.showDialog = true;
        },

        /**
         * Method d??ng ????? ????nh d???u l?? th??m s???n ph???m
         * CreatedBy : NXHoang 
         */
        setIsProduct()
        {
            this.isAddOrEdit = Enum.FormModel.Add;
        }


    },

    created() {
        this.getData();
        this.getProductCategories();
    },
}
</script>

<style>

</style>