<template>
  <card-component title="VIP Launcher" icon="lock" :has-card-header-background="true" :has-button-slot="true">

    <!--
    <router-link v-if="is_authed" slot="button" to="https://discord.gg/tkr45WA" class="button is-small">
      Discord
    </router-link>
    -->

    <div v-if="!is_authed">

        <b-field label="Serial">
          <b-input type="textarea" v-model="form.serial" placeholder="" rows="2" maxlength="255" required autofocus/>
        </b-field>
  <!--
        <hr>

        
          <div class="field">
            <b-checkbox v-model="form.opt1">Hardware spoof</b-checkbox>
          </div> 
          -->


          <b-field label="Spoofer">
              <b-select placeholder="Spoofer Type" v-model="form.opt1">
                  <option v-for="option in spoofers" :value="option.id" :key="option.type">
                      {{ option.text }}
                  </option>
              </b-select>
          </b-field>

          <!--
          <div class="field">
            <b-checkbox v-model="form.opt2">Legacy mode <span style="font-family: monospace">( windows &lt;=1709 )</span></b-checkbox>
          </div>      
          -->
        <hr>

        <b-field grouped>
          <div class="control">
            <button type="submit" class="button is-black" :class="{'is-loading':is_loading}" @click="authenticate">
              Authenticate
            </button>
          </div>
          <!--
          <div class="control">
            <router-link to="/password-recovery" class="button is-outlined is-black">
              Help
            </router-link>
          </div>
          -->
        </b-field>
    </div>

    <!-- loader dialog -->

    <div v-else>
      <div class="media-content">
        <p class="title is-4">{{ launch_dialog.name }} Subscription</p>
      </div>
      <hr>
        <article class="message is-warning"><!----> <section class="message-body"><div class="media"><div class="media-left"><span class="icon is-large is-warning"><i class="mdi mdi-alert mdi-48px"></i></span></div> <div class="media-content">
           Remember to always reboot after using product. Never run the spoofer more than once per boot.
        </div></div></section></article>

      <div class="content">
        <ul>
          <li>Your subscription expires {{ launch_dialog.time_end }}</li>
          <li>Cheat Last Update: {{ launch_dialog.time_update }}</li>
          <li>Test Build Access: False</li>
        </ul>
      </div>

      <b-field grouped>
        <div class="control">
            <button type="submit" class="button is-black" @click="launch">
              Launch!
            </button>
        </div>   
        <div class="control">
            <button class="button is-black is-outlined" @click="close_launch">
              Back
            </button>
        </div>
      </b-field>


    </div>

  </card-component>
</template>

<script>
/* eslint-disable */ 
import CardComponent from '@/components/CardComponent'

export default {
  name: 'Login',
  components: { CardComponent },
  data () {
    return {
      dialog: "auth",
      is_loading: false,
      is_authed: false,
      spoofer_type: 0,
      spoofers: [
        { id: 0, text: 'No Spoofer' },
        { id: 1, text: 'BE Spoofer' },
        { id: 2, text: 'EAC Spoofer' },
      ],
      form: {
        serial: "",
        action: false,
        opt1: 1,
        opt2: false,
      },
      launch_dialog: {
        access_key: "",
        beta_available: 0,
        name: "",
        time_end: "",
        time_update: "",
        base_url: "https://atlascheats.com/api/auth/gateway.php?serial=",
      }
    }
  },
  methods: {
    authenticate( )
    {
        this.is_loading = true;
        console.log( 'connecting' );
        console.log( 'opt1', this.form.opt1 );

        this.form.action = "auth";
        this.$http.post( 'https://atlascheats.com/api/auth/gateway.php', this.form ).then( function( response )
        {
          console.log( 'response', response );

          //console.log('pollResult( )', this.$root.serverRequest);
          //this.loading = false;
          //this.queryResult = response.data;

          if ( response.data.code != 200 )
          {
              this.$buefy.snackbar.open( { message: response.body.error, queue: false } )             
              this.is_loading = false;
              return false;
          }
         
          this.launch_dialog.name = response.data.result.name;
          this.launch_dialog.time_end = response.data.result.time_end;
          this.launch_dialog.time_update = response.data.result.time_updated;

          this.is_loading = false;
          this.is_authed  = true;


          return true;

        }, function ( response ) {

            this.$buefy.snackbar.open( { message: "Service temporarily unavailable", queue: false } )
            this.is_loading = false;
            return false;

        });
    },    
    launch( )
    {
        //this.form.opt1 = this.spoofer_type;

        var download_url
        = this.launch_dialog.base_url.concat(
            this.form.serial,
            "&action=serve_loader",
            "&opt1=",
            ~~this.form.opt1,
            "&opt2=",
            ~~this.form.opt2 );

        location.replace( download_url );
    },
    close_launch( )
    {
        this.is_authed = false;
    }
  }
}
</script>
