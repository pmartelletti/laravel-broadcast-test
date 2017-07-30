<template>
    <div class="container">
        <div class="row">
            <div class="col-md-8 col-md-offset-2">
                <div class="panel-default panel" v-for="message in messages">
                    <div class="panel-heading">
                        <h3 class="panel-title text-right">Said {{ ago(message) }}</h3>
                    </div>
                    <div class="panel-body">
                        <p>Said: {{ message.message }}</p>
                    </div>
                </div>
            </div>

            <div class="alert alert-info" v-if="typing">
                <p>Someone is typing...</p>
            </div>

            <div class="col-md-8 col-md-offset-2">
                <div class="panel-default panel">
                    <div class="panel-heading">
                        <h3 class="panel-title text-right">New Message</h3>
                    </div>
                    <div class="panel-body">
                        <form @submit.prevent="sendMessage">
                            <div class="form-group">
                                <label>Message</label>
                                <textarea @keyup="typingHint" class="form-control" v-model="form.message" ref='form' ></textarea>
                            </div>
                            <button type="submit" class="btn btn-default">Add</button>

                        </form>
                    </div>
                </div>
            </div>

        </div>
    </div>
</template>

<script>
    import moment from 'moment';
    export default {
        data() {
            return {
                messages: [],
                typing: false,
                form: {
                    message: ''
                }
            }
        },
        mounted() {
            this.$refs.form.focus();
            this.getMessages()
            Echo.channel('messages').listenForWhisper('typing', () => {
                this.typing = true;
            })
            Echo.channel('messages')
                    .listen('MessageWasAdded', (e) => {
                        this.typing = false;
                        this.messages.push(e.message)
                    });
        },
        methods: {
            ago(message) {
                return moment(message.created_at).fromNow() + '...';
            },
            getMessages() {
                axios.get('/api/messages')
                        .then(({data}) => this.messages = data)
            },
            typingHint() {
                Echo.channel('messages')
                        .whisper('typing', {});
            },
            sendMessage() {
                axios.post('/api/messages', {'message': this.form.message})
                        .then(({data}) => this.reloadData(data))
            },
            reloadData(message) {
                this.form.message = ''
//                this.getMessages()
                this.messages.push(message)
                this.$refs.form.focus();
            }
        }
    }
</script>
