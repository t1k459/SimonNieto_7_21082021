<template>
     <div>
        <Header />
        <div class="container">
            <div class="row justify-content-center">
                <div class="col-12 col-md-10 col-lg-8" id="allMessages">
                   <b-button @click="$bvModal.show('modalAddMessage')" href="" data-toggle="modal" data-target="#modalAddMessage" class="my-2 btn btn-sm btn-block " >Poster un message...</b-button>
                    <b-modal hide-footer hide-header class="modal fade" id="modalAddMessage" tabindex="-1" aria-labelledby="modalAddMessage" aria-hidden="true" :ok-disabled="true">
                        <div class="modal-dialog">
                            <div class="modal-content">
                                <form enctype="multipart/form-data">
                                    <div class="modal-header">
                                        <p class="modal-title h5">Poster un nouveau message</p>
                                    </div>
                                    <div class="row modal-body">
                                        <div class="col-12 justify-content-center form-group">
                                            <label for="newMessage" class="sr-only">Message :</label>
                                            <textarea class="form-control" v-model="newMessage" id="newMessage" name="message" rows="10" placeholder="Votre message ici..."></textarea>
                                        </div>
                                        <div class="col-12 justify-content-center text-center">
                                            <img :src="newImage" class="w-50 rounded">
                                            <p class="small text-center">Image à partager</p>
                                        </div>
                                        <div class="col-12 justify-content-center">
                                            <div class="form-group justify-content-center">
                                                <label for="File" class="sr-only">Choisir une nouvelle photo</label>
                                                <input @change="onFileChange()" type="file" ref="file" name="image" class="form-control-file" id="File" accept=".jpg, .jpeg, .gif, .png">
                                            </div>
                                        </div>
                                    </div>
                                    <div class="modal-footer">
                                        <div class="row w-100 justify-content-spacebetween">
                                            <div class="col-6"><b-button id="closemodal" @click="$bvModal.hide('modalAddMessage')" data-dismiss="modal" class="btn btn-secondary btn-block">Annuler</b-button></div>
                                            <div class="col-6"><button type="submit" @click.prevent="addNewMessage()" class="btn text-light bg-dark btn-block">Valider</button></div>
                                        </div>
                                    </div>
                                </form>
                            </div>
                        </div>
                    </b-modal>
                <div>

                </div>
                    <div v-for="message in messages" :key="message.id" class="card border bg-light my-3">
                        <div class="card-header bg-light d-flex align-items-center justify-content-between m-0 p-1">
                            <div>
                                <img :src="message.avatar" alt="image de profil" height="40" class="m-0 rounded-circle"/>
                                <span class="small text-dark m-0 p-1" >
                                    Posté par {{message.userName}} 
                                    <span v-if="!message.isActive" class="small text-danger">(supprimé)</span>, 
                                    le {{message.createdAt.slice(0,10).split('-').reverse().join('/') + ' à ' + message.createdAt.slice(11,16)}}
                                </span>
                            </div>                                
                            <div v-if="isAdmin == 'true'">

                                    <a :href="'/message/drop/' + message.id"><img src="/images/drop.svg" class="m-1 p-0" alt="Supprimer le message" title="Supprimer le message"/></a>
                            </div>
                            <div v-if="(message.UserId == currentUserId) && (isAdmin == 'false') ">
                                    <a :href="'/message/edit/' + message.id"><img src="/images/edit.svg" class="m-1 p-0" alt="Editer le message" title="Editer le message"/></a>
                                    <a :href="'/message/drop/' + message.id"><img src="/images/drop.svg" class="m-1 p-0" alt="Supprimer le message" title="Supprimer le message"/></a>
                            </div>                              
                        </div>
                        <div class="card-body text-dark text-left">
                            <p class="small" v-if="message.message !== ''"> {{message.message}} </p>
                            <img class="w-100" :src="message.messageUrl" alt="image du message" v-if="message.messageUrl !== ''">
                        </div>
                        <div class="card-footer bg-light text-dark text-left m-0">
                            <a :href="'/commentaires/' + message.id" class="h6 text-dark">Voir les commentaires</a>
                        </div>
                    </div>
                    <noMessage v-if="noMessage"></noMessage>
                    
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import Header from "./Header.vue"
import NoMessage from "./NoMessage.vue"
import Swal from "sweetalert2"
import axios from "axios"

export default {
    name: "Home",
    components:{
        Header,
        NoMessage
    },
    data() {
        return {
            noMessage: false,
            isAdmin: false,
            isActive: true,
            newImage: "",
            currentUserId: "", 
            newMessage: "",
            file: null,
            messages: [],
        }
    },
    methods: {
        onFileChange() {
            this.file = this.$refs.file.files[0];
            this.newImage = URL.createObjectURL(this.file)
        },
        addNewMessage() {
            const formData = new FormData()
            formData.set("image", this.file)
            formData.set("UserId", this.currentUserId.toString())
            formData.set("message", this.newMessage.toString())
            axios.post("http://127.0.0.1:3000/api/messages/", formData, { headers: { "Authorization":"Bearer " + localStorage.getItem("token")}})
            .then(()=> {
                this.UserId = ""
                this.newMessage = ""
                this.file = null
                Swal.fire({
                    text: "Message posté !",
                    footer: "Redirection en cours...",
                    icon: "success",
                    timer: 2000,
                    showConfirmButton: false,
                    timerProgressBar: true,
                    willClose: () => { location.reload() }
                })
            })
            .catch((error)=>{
                const codeError = error.message.split("code ")[1]
                let messageError = ""
                switch (codeError){
                    case "400": messageError = "Le message n'a pas été posté !"; break
                    case "401": messageError = "Requête .. non-authentifiée !"; break
                }
                Swal.fire({
                    title: "Une erreur est survenue",
                    text: messageError || error.message,
                    icon: "error",
                    timer: 3500,
                    showConfirmButton: false,
                    timerProgressBar: true
                }) 
            })
        }
    },
    created: function() {
        this.isAdmin = localStorage.getItem("role")
        this.currentUserId = localStorage.getItem("userId")
        if (localStorage.getItem("refresh")===null) {
            localStorage.setItem("refresh", 0)
            location.reload()
        }
        axios.get("http://127.0.0.1:3000/api/messages",{ headers: {"Authorization": "Bearer " + localStorage.getItem("token")} })
        .then(res => {
            const rep = res.data.ListeMessages
            if (rep.length === 0) { this.noMessage = true } else { this.noMessage = false }
            this.messages = rep
        })
        .catch((error)=>{
            const codeError = error.message.split("code ")[1]
            let messageError = ""
            switch (codeError){
                case "400": messageError = "La liste des messages n'a pas été récupérer !"; break
            }
            Swal.fire({
                title: "Une erreur est survenue",
                text: messageError || error.message,
                icon: "error",
                timer: 3500,
                showConfirmButton: false,
                timerProgressBar: true
            }) 
        })
    }
}
</script>

<style>

</style>