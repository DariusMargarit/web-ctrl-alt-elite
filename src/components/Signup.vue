<template>
    <v-dialog class="basePage" width="500" v-model="dialogSignUp">
        <template v-slot:activator="{ on }">
            <v-btn plain v-on="on">Sign Up</v-btn>
        </template>
        <v-card>
            <link rel="preconnect" href="https://fonts.gstatic.com">
            <link href="https://fonts.googleapis.com/css2?family=Archivo:wght@300&family=Arvo&family=Crimson+Text&family=Exo:wght@100&family=Sriracha&family=Tenor+Sans&display=swap" rel="stylesheet">

            <form class="formdesign" @submit.prevent="submitForm">
                <div @click="dialogSignUp = !dialogSignUp"><v-icon class="close">mdi-close</v-icon></div>
                <h1>Sign up!</h1>
                <br />
                <hr />
                <br />
                <div>
                    <span class="text">Please sign up with an existing account</span>
                </div>
                <br />
                <div class="wrapper">
                    <meta name="viewport" content="width=device-width, initial-scale=1">
                    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/4.7.0/css/font-awesome.min.css">
                    <div class="button">
                        <div class="icon-facebook" @click="facebookSignUp">
                            <i><v-icon class="pb-1" size="40" >mdi-facebook</v-icon></i>
                        </div>
                    </div>
                    <div class="button" @click="googleSignUp">
                        <div class="icon-google">
                            <i><v-icon class="pb-1" size="35">mdi-google</v-icon></i>
                        </div>
                    </div>
                </div>
                <br />
                <div class="separator text">or create a new one</div>
                <br />
                <div align="center">
                    <v-text-field
                            v-model="name"
                            :error-messages="nameErrors"
                            :counter="15"
                            label="Username"
                            required
                            @input="$v.name.$touch()"
                            @blur="$v.name.$touch()">
                    </v-text-field></div>

                <v-text-field
                        v-model="email"
                        :error-messages="emailErrors"
                        label="E-mail"
                        required
                        @input="$v.email.$touch()"
                        @blur="$v.email.$touch()">
                </v-text-field>

                <v-text-field
                        label="Password"
                        type="password"
                        v-model="password"
                        :error-messages="passwordErrors"
                        required
                        @input="$v.password.$touch()"
                        @blur="$v.password.$touch()">
                </v-text-field>
                <br />
                <div align="center">
                    <v-btn class="mr-4" type="submit" :loading="loading">submit</v-btn>
                    <v-btn class="bttn" @click="clear">clear</v-btn>
                </div>
                <br />
                <br />
                <hr class="text"/>
                <div class="text">
                    <span>Already have an account?</span>
                    <v-btn class="ml-3" text float="right"
                           @click="dialogSignUp = !dialogSignUp">Log in</v-btn>
                </div>
            </form>
        </v-card>

    </v-dialog>
</template>

<script>
    import { validationMixin } from 'vuelidate'
    import { required, maxLength, email, minLength } from 'vuelidate/lib/validators'
    import firebase from 'firebase/app'
    import 'firebase/auth'
    import 'firebase/firestore'
    export default {
        name: "signup",
        mixins: [validationMixin],
        validations: {
            name: { required, maxLength: maxLength(15), minLength: minLength(3) },
            email: { required, email },
            password: { required, minLength: minLength(6) }
        },
        data: () => ({
            name: '',
            email: '',
            password: '',
            loading: false,
            dialogSignUp: false
        }),
        computed: {
            nameErrors () {
                const errors = []
                if (!this.$v.name.$dirty) return errors
                !this.$v.name.maxLength && errors.push('Name must be at most 15 characters long')
                !this.$v.name.required && errors.push('Name is required')
                !this.$v.name.minLength && errors.push('Name must be at least 3 characters long')
                return errors
            },
            emailErrors () {
                const errors = []
                if (!this.$v.email.$dirty) return errors
                !this.$v.email.email && errors.push('Must be valid e-mail')
                !this.$v.email.required && errors.push('E-mail is required')
                return errors
            },
            passwordErrors () {
                const errors = []
                if(!this.$v.password.$dirty) return errors
                !this.$v.password.required && errors.push('Password is required')
                !this.$v.password.minLength && errors.push('Password must be at least 6 characters long')
                return errors
            },
        },
        methods: {
            submitForm () {
                this.loading = true;
                firebase.auth().createUserWithEmailAndPassword(this.email, this.password).then(cred => {
                    return firebase.firestore().collection('users').doc(cred.user.uid).set({
                        username: this.name
                    });
                }).then(() => {
                    this.dialogSignUp = false;
                    this.loading = false;
                }).catch((err) => {
                    console.log(err)
                });
            },
            googleSignUp () {
                var provider = new firebase.auth.GoogleAuthProvider();
                firebase.auth().signInWithPopup(provider).then((res) => {
                    return firebase.firestore().collection('users').doc(res.user.uid).set({
                        username: res.additionalUserInfo.profile.name
                    }).then(() => {
                        console.log('Success sign up with Google')
                        this.dialogSignUp = false;

                    });
                }).catch((err) => {
                    console.log(err)
                    console.log('Failed to sign up with Gooogle')
                });
            },
            facebookSignUp () {
                var provider = new firebase.auth.FacebookAuthProvider();
                firebase.auth().signInWithPopup(provider).then((res) => {
                    return firebase.firestore().collection('users').doc(res.user.uid).set({
                        username: res.additionalUserInfo.profile.name
                    }).then(() => {
                        console.log(res)
                        console.log('Success sign up with Facebook')
                        this.dialogSignUp = false;
                    });
                }).catch((err) => {
                    console.log(err)
                    console.log('Failed to sign up with Facebook')
                })
            },
            clear () {
                this.$v.$reset()
                this.name = ''
                this.email = ''
                this.password = ''
                this.loading = false
            },
        },
    }
</script>

<style scoped>
    @import url('https://fonts.googleapis.com/css2?family=Crimson+Text&display=swap');
    .basePage {
        justify-content: center;
        background-color: floralwhite;
        font-family: 'Tenor Sans', sans-serif;
    }
    .formdesign {
        align: center;
        text-align: center;
        background-color: ghostwhite;
        padding: 60px;
        font-family: 'Arvo', serif;
    }
    .bttn {
        margin-left: 6px;
    }
    .text {
        color: gray;
        font-family: 'Arvo', serif;
    }
    .separator {
        display: flex;
        align-items: center;
        text-align: center;
        font-family: 'Arvo', serif;
    }
    .separator::before,
    .separator::after {
        content: '';
        flex: 1;
        border-bottom: 1px solid gray;
    }
    .separator:not(:empty)::before {
        margin-right: .25em;
    }
    .separator:not(:empty)::after {
        margin-left: .25em;
    }
    .wrapper .button {
        display: inline-block;
        height: 60px;
        width: 60px;
        align-content: center;
        margin: 0 5px;
        overflow: hidden;
        background: #fff;
        border-radius: 50px;
        cursor: pointer;
        box-shadow: 0px 10px 10px rgba(0,0,0,0.1);
    }
    .wrapper .button {
        font-size: 20px;
        font-weight: 500;
        line-height: 60px;
        margin-left: 10px;
    }
    .wrapper .button {
        font-size: 25px;
        line-height: 60px;
    }
    .wrapper .button:hover .icon-facebook{
        background: rgb(54, 79, 243);
    }
    .wrapper .button:hover .icon-google{
        background: rgb(253, 59, 31);
    }
    .wrapper .button:hover .icon-facebook i{
        color: #fff;
    }
    .wrapper .button:hover .icon-google i {
        color: #fff;
    }
    .close {
        align:right;
        float: right;
    }
    .close:hover {
        cursor:pointer;
        background-color: #e3e3fd;
    }
</style>