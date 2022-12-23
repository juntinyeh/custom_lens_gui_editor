<template>
  <v-app id="inspire">
    <v-system-bar app class="py-6" color="blue lighten-2">
      <v-col cols="4">
      <v-avatar
          color="white darken-1"
          size="36"
        ><v-icon>mdi-code-json</v-icon></v-avatar>
        &nbsp; <b>Cuctom Lens Editor</b>
      </v-col>
      <v-col>
        <v-file-input
          v-model="inputFile"
          @change="readFile"
          placeholder="import JSON File"
        ></v-file-input>
        </v-col>
        <v-col align="center">          
        <v-btn
            v-if="inputFile"
            icon
            @click="dialog_preview = true"
          >
            <v-icon>mdi-eye</v-icon>
            Preview
          </v-btn>
      </v-col>

    </v-system-bar>

    <v-navigation-drawer
      app
    >
      <v-divider></v-divider>

      <v-list>
        <v-list-item
          v-for="[icon, pillar_id, pillar_name] in links"
          :key="pillar_id"
          link
        >
          <v-list-item-icon>
            <v-icon>{{ icon }}</v-icon>
          </v-list-item-icon>

          <v-list-item-content>
            <v-list-item-title              
              @click="v_load_pillar(pillar_id, pillar_name)"
            >{{ pillar_name }}</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>

    <v-main>
      <v-container
        class="py-8 px-6"
        fluid
      >
        <v-row>
          <v-col>
            <v-card
              class="overflow-auto"
              >
              <v-subheader>
                <v-icon>mdi-pillar</v-icon>
                <h1>{{ pillar_title }}</h1>
              </v-subheader>
              <v-list two-line
                v-if="current_pillar"
                >
                <template>
                  <v-expansion-panels focusable>
                    <v-expansion-panel
                      v-for="q in questions"
                      :key="q.id"
                      >
                      <v-expansion-panel-header>
                        <div>
                        <v-icon>mdi-comment-question</v-icon>
                        ({{ q.id }})
                        {{ q.title }}
                        </div>
                      </v-expansion-panel-header>
                      <v-expansion-panel-content>
                        <div
                          v-for="qc in q.choices"
                          :key="qc.id"
                        >
                          <div inline>
                            <v-icon
                              small>mdi-card-outline</v-icon>
                            ({{ qc.id }}) &nbsp; {{qc.title}} &nbsp;
                            <v-icon 
                              @click="v_form_choice(q.id, qc.id)"
                              small>
                              mdi-pencil
                            </v-icon>
                          </div>
                        </div>
                        <div align="right">
                          <v-btn
                            @click="v_form_choice(q.id, 'cl_create_new_choice')"
                            color="green darken"
                            medium
                            > <v-icon
                           small>mdi-card-plus</v-icon> Add a new choice to this question ....</v-btn>
                        </div>
                        <hr />
                        <div>
                          <b>riskRules</b>
                          <br />
                          {{q.riskRules}}
                          <br />
                          <v-icon 
                              @click="v_form_question(q.id)"
                              small>
                              mdi-pencil
                            </v-icon>
                        </div>
                      </v-expansion-panel-content>
                    </v-expansion-panel>
                  </v-expansion-panels>
                  <div align="right">
                          <v-btn
                            @click="v_form_question('cl_create_new_question')"
                            color="blue lighten-3"
                            medium
                            > <v-icon
                           small>mdi-card-plus</v-icon> Add a new question ....</v-btn>
                        </div>
                </template>
              </v-list>
            </v-card>
            </v-col>
          </v-row>          
        <v-row justify="center">
          <v-dialog
            v-model="dialog"
            persistent
            max-width="600px"
          >            
            <v-card>
              <v-card-title>
                <span class="text-h5">Choice</span>
              </v-card-title>
              <v-card-text>
                <v-container>
                  <v-row>
                    <v-col cols="4">
                      <v-text-field
                        v-model="current_choice.id"
                        label="Choice Unique Id">
                      </v-text-field>
                    </v-col>
                    <v-col
                      cols="12"
                    >
                      <v-text-field
                        v-model="current_choice.title"
                        label="Choice Title"
                        required
                      ></v-text-field>
                    </v-col>
                    <v-col
                      cols="12"
                    >
                      <v-text-field
                        v-model="current_choice.description"
                        label="Choice Description"
                        hint="example of helper text only on focus"
                      ></v-text-field>
                    </v-col>
                  </v-row>
                  <v-row>
                    <v-col cols="12">
                      <h2>
                        Helpful Resource
                      </h2>
                    </v-col>
                  </v-row>
                  <v-row>
                    <v-col cols="12">
                      <v-text-field
                        v-model="current_choice.helpfulResource.displayText"
                        label="Display Text"                        
                        hint="example of helper text only on focus"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12">
                      <v-text-field
                        v-model="current_choice.helpfulResource.url"
                        label="Reference URL"                        
                        hint="example of helper text only on focus"
                      ></v-text-field>
                    </v-col>
                  </v-row>
                  <v-row>
                    <v-col cols="12">
                      <h2>
                        Improvement Plan
                      </h2>
                    </v-col>
                  </v-row>
                  <v-row>
                    <v-col cols="12">
                      <v-text-field
                        v-model="current_choice.improvementPlan.displayText"
                        label="Display Text"
                        hint="example of helper text only on focus"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12">
                      <v-text-field
                        v-model="current_choice.improvementPlan.url"
                        label="Reference URL"
                        hint="example of helper text only on focus"
                      ></v-text-field>
                    </v-col>
                  </v-row>                  

                </v-container>
              </v-card-text>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn
                  color="blue darken-1"
                  text
                  @click="dialog = false"
                >
                  Close
                </v-btn>
                <v-btn
                  color="blue darken-1"
                  text
                  @click="dialog = false;update_current_choice()"
                >
                  Save
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-row>

        <v-row justify="center">
          <v-dialog
            v-model="dialog_risk"
            persistent
            max-width="600px"
          >            
            <v-card>
              <v-card-title>
                <span class="text-h5">Question</span>
              </v-card-title>
              <v-card-text>
                <v-container>
                  <v-row>
                    <v-col cols="12">
                      <v-text-field
                        v-model="current_question.id"
                        label="Question Unique ID"
                        required
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12">
                      <v-text-field
                        v-model="current_question.title"
                        label="Question Title"
                        required
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12">
                      <v-text-field
                        v-model="current_question.description"
                        label="Question Description"
                        required
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12">
                      <v-text-field
                        v-model="c_riskrule"                        
                        label="RiskRules"
                        required
                      ></v-text-field>
                    </v-col>
                  </v-row>                  

                </v-container>
              </v-card-text>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn
                  color="blue darken-1"
                  text
                  @click="dialog_risk = false"
                >
                  Close
                </v-btn>
                <v-btn
                  color="blue darken-1"
                  text
                  @click="dialog_risk = false;update_current_question()"
                >
                  Save
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-row>

        <v-row justify="center">
    <v-dialog
      v-model="dialog_preview"
      fullscreen
      hide-overlay
      transition="dialog-bottom-transition"
    >
      <v-card>
        <v-toolbar
          dark
          color="primary"
        >
          <v-btn
            icon
            dark
            @click="dialog_preview = false"
          >
            <v-icon>mdi-close</v-icon>
          </v-btn>
          <v-toolbar-title>Custom Lens Preview</v-toolbar-title>
          <v-spacer></v-spacer>
          <v-toolbar-items>
            <v-btn
              dark
              text
              @click="saveFile()"
            >
              Save
            </v-btn>
          </v-toolbar-items>
        </v-toolbar>
        <div><pre>{{ JSON.stringify(this.custom_lens,'',2) }}</pre></div>
      </v-card>
    </v-dialog>
  </v-row>




      </v-container>
    </v-main>
  </v-app>

</template>
<script>
  export default {
    data: () => ({
      pillar_title: 'Welcome to Well-Architected Custom Lens JSON Editor',
      inputFile: null,
      links: [
        ['mdi-numeric-1', 'Pillar_1', 'Import a JSON file'],
        ['mdi-numeric-2', 'Pillar_2', 'Click Pillar'],
        ['mdi-numeric-3', 'Pillar_3', 'Click Question'],
        ['mdi-numeric-4', 'Pillar_4', 'Update'],
        ['mdi-numeric-5', 'Pillar_5', 'Preview/Save'],
      ],
      questions: [
        ['question_id','Question Title'],
      ],
      custom_lens : null,
      current_pillar : null,
      current_question : null , 
      current_choice: null,      
      dialog : false,
      dialog_risk : false,
      dialog_preview : false,
      c_riskrule : '',
    }),

    methods: {
      readFile: function() {        
        const reader = new FileReader();
        if (this.inputFile.name.includes(".json")) {
          reader.onload = (e) => {
            if(this.valid_cl(e.target.result)){
              this.custom_lens = JSON.parse(e.target.result);
              this.custom_lens.ready = true;
              this.v_update_pillars();
            }
          }
          reader.onerror = (err) => console.log(err);
          reader.readAsText(this.inputFile);        
        }
        this.reset_current_choice();
        this.reset_current_question();
      },

      valid_cl: function(input_cl){
        try {
            JSON.parse(input_cl);
        }
        catch(e) {
            this.err = JSON.stringify(e.message);
            console.log(this.err);            
            return "";
        }
        return true;
      }, 

      cl_get_pillars: function(){
        if(typeof(this.custom_lens.ready) !=  'undefined'){
          if(typeof(this.custom_lens.pillars) != 'undefined')
            return this.custom_lens.pillars;
        }
      },

      v_update_pillars: function(){
        this.links = [];
        this.custom_lens.pillars.forEach((p) => {         
          this.links.push(['mdi-pillar', p.id, p.name]);
        });
      },

      v_load_pillar: function(pillar_id, pillar_name){
        this.pillar_title = pillar_name;        
        this.custom_lens.pillars.forEach((p) => {
          if(p.id==pillar_id){ this.current_pillar = p; }
        });
        this.questions = [];
        this.current_pillar.questions.forEach((q) => {
          this.questions.push(q);
        });
      },

      v_reload_pillar: function() {
        this.questions = [];
        this.current_pillar.questions.forEach((q) => {
          this.questions.push(q);
        });
      },

      v_form_choice: function(question_id, choice_id){
        if(choice_id=='cl_create_new_choice'){this.reset_current_choice()}
        this.questions.forEach((q) => {
          if(q.id == question_id){
            this.current_question = q;
          }
        });
        this.current_question.choices.forEach((c) => {
          if(c.id == choice_id){
            this.current_choice = c;
          }
        });
        this.dialog = true;
      },

      v_form_question: function(question_id){
        if(question_id=='cl_create_new_question'){this.reset_current_question()}
        this.questions.forEach((q) => {
          if(q.id == question_id){
            this.current_question = q;
          }
        });
        this.c_riskrule = JSON.stringify(this.current_question.riskRules)
        this.dialog_risk = true;
      },


      update_current_choice(){
        var target_c = -1;
        var target_q = -1;
        var target_p = -1;
        this.current_question.choices.forEach((c,i) => {
          if(c.id == this.current_choice.id){
              target_c = i;
            }
          });
        this.current_pillar.questions.forEach((q,i) => {
          if(q.id == this.current_question.id){
            target_q = i;
          }
        });  
        this.custom_lens.pillars.forEach((p,i) => {
          if(p.id == this.current_pillar.id){
            target_p = i;
          }
        });

        if(target_q >=0 && target_p >= 0){
          if(target_c >=0 ){
            this.custom_lens.pillars[target_p].questions[target_q].choices[target_c] = this.current_choice;
          }else
          {
            this.custom_lens.pillars[target_p].questions[target_q].choices.push(this.current_choice);
          }
        }      
        this.v_reload_pillar();
      },

      update_current_question(){
        var target_q = -1;
        var target_p = -1;
        var obj;
        try{
          obj = JSON.parse(this.c_riskrule);
          this.current_question.riskRules = obj;
        }catch(err)
        {
          alert("Invalid format for riskRules : " + err.message);
        }
        this.current_pillar.questions.forEach((q,i) => {
          if(q.id == this.current_question.id){
            target_q = i;
          }
        });  
        this.custom_lens.pillars.forEach((p,i) => {
          if(p.id == this.current_pillar.id){
            target_p = i;
          }
        });
        if( target_p >= 0 ){
          if (target_q >= 0 ){
            this.custom_lens.pillars[target_p].questions[target_q] = this.current_question;
          }else{
            this.reset_current_choice();
            this.current_question.choices = [];
            this.current_question.choices.push(this.current_choice);
            this.custom_lens.pillars[target_p].questions.push(this.current_question);
          }
        }
        this.v_reload_pillar();
      },


      saveFile() {
        const data = JSON.stringify(this.custom_lens)
        const blob = new Blob([data], {type: 'text/plain'})
        const e = document.createEvent('MouseEvents'),
        a = document.createElement('a');
        a.download = this.custom_lens.name+ "-" +(Math.random().toString(36).slice(2))+".json";
        a.href = window.URL.createObjectURL(blob);
        a.dataset.downloadurl = ['text/json', a.download, a.href].join(':');
        e.initEvent('click', true, false, window, 0, 0, 0, 0, 0, false, false, false, false, 0, null);
        a.dispatchEvent(e);
      },

      reset_current_choice() {
        this.current_choice = { 
          id :0,
          title : "choice_title",
          description : "choice_description",
          helpfulResource : {
            displayText : "Text for helpfulResource", 
            url : "url for helpfulResource",
          },
          improvementPlan : {
            displayText : "Text for improvementPlan", 
            url : "url for improvementPlan",
          }
        };
      },

      reset_current_question() {
        this.current_question = { 
        id: 0,
        title : "question_title",
        description : "choice_description",
        riskRules : [{
                     "condition":"(!choice_key1) || choice_key2",
                     "risk":"HIGH_RISK"
                  },
                  {
                     "condition":"default",
                     "risk":"MEDIUM_RISK"
                  }],
        }
      }
    },
    beforeMount(){
    this.reset_current_choice();
    this.reset_current_question();
    }
  }
</script>