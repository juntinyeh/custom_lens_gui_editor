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
        <v-btn            
            text
            @click="load_template()"
          >Use Template
        </v-btn>
      </v-col>
      <v-col>
        <v-file-input
          v-if="!inputFile"
          v-model="inputFile"
          @change="readFile"
          placeholder="import JSON File"
        ></v-file-input>
        </v-col>
        <v-col align="center">          
        <v-btn
            v-if="custom_lens"
            icon
            @click="dialog_preview = true"
          >
            <v-icon>mdi-eye</v-icon>
            Preview
        </v-btn>
      </v-col>
      <v-col cols="1">
        <a href="https://github.com/juntinyeh/custom_lens_gui_editor/tree/main"><v-icon>mdi-github</v-icon></a>
      </v-col>

    </v-system-bar>

    <v-navigation-drawer
      app
    >
      
      <v-list>
        <v-list-item
          v-for="[icon, pillar_id, pillar_name] in links"
          :key="pillar_id"
          link>
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
      <v-divider></v-divider>
      <v-list>
        <v-list-item>
          <h5>Resources</h5>
        </v-list-item>
        <v-list-item>
          <a href="https://www.wellarchitectedlabs.com/well-architectedtool/100_labs/100_custom_lenses_on_watool/">Well-Architected Labs - Custom Lens on WA-Tool</a>
        </v-list-item>
        <v-list-item>
          <a href="https://docs.aws.amazon.com/wellarchitected/latest/userguide/lenses-custom.html">Official User Guide for Custom Lenses</a>
        </v-list-item>
        <v-list-item>
          <a href="https://docs.aws.amazon.com/wellarchitected/latest/userguide/lenses-format-specification.html">Lens format specification</a>
        </v-list-item>
        <v-list-item>
          <a href="https://aws.amazon.com/blogs/aws/well-architected-custom-lenses-internal-best-practices/">Blog: Well-Architected Custom Lenses for Internal Best Practices</a>
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
                            </v-icon> &nbsp;                            
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
                <div>
                <span class="text-h5">Choice</span>
                </div>
                <div inline align="right">
                <v-btn
                      color="blue darken-1"
                      text
                      @click="dialog = false"
                    >
                      <v-icon>mdi-close</v-icon>
                      
                    </v-btn>
                  </div>
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
                <v-row>
                  <v-col col="1"
                  align="left">
                    <v-btn
                      v-if="current_choice.id != ''"
                      color="red"
                      @click="delete_type = 'choice'; v_form_delete(current_question.id, null)"
                      >
                      Delete
                    </v-btn>
                  </v-col>
                  <v-col  col="1">
                    <v-btn
                      color="blue"
                      @click="dialog = false"
                    >
                    Close
                    </v-btn>
                  </v-col>
                  <v-col  col="1">
                    <v-btn
                      v-if="current_choice.id != ''"
                      color="green"
                      @click="dialog = false;update_current_choice()"
                    >
                    Update
                    </v-btn>
                </v-col>
              </v-row>
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
                <v-row>
                  <v-col col="1"
                  align="left">
                    <v-btn
                      v-if="current_question.id != ''"
                      color="red"
                      @click="delete_type = 'question'; v_form_delete(current_question.id, null)"
                      >
                      Delete
                    </v-btn>
                  </v-col>
                  <v-col  col="1">
                    <v-btn
                      color="blue"
                      @click="dialog_risk = false"
                    >
                      Close
                    </v-btn>
                  </v-col>
                  <v-col  col="1">
                    <v-btn
                    v-if="current_question.id != ''"
                      color="green"
                      @click="dialog_risk = false;update_current_question()"
                    >
                      Update
                    </v-btn>
                  </v-col>
                </v-row>
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

        <v-row justify="center">
          <v-dialog
            v-model="dialog_delete"
            persistent
            max-width="400px"
          >            
            <v-card>
              <v-card-title>
                <span class="text-h5">Delete?</span>
              </v-card-title>
              <v-card-text>
                <v-container>
                  <v-row>
                    <v-col cols="12">
                      <div>
                        <h4>Please make sure you are going to remove this item from custom lens structure: {{ delete_message }}
                        </h4></div>
                    </v-col>
                    <v-col cols="12">
                      <v-text-field
                        v-model="delete_confirm"                        
                        required
                        label="Type in 'Delete' to confirm."
                        placeholder="Delete"
                      ></v-text-field>
                    </v-col>
                  </v-row>                  
                </v-container>
              </v-card-text>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn
                  v-if="delete_confirm=='Delete'"
                  color="red darken-1"
                  @click="dialog_delete = false; dialog_risk = false; dialog = false;delete_current_object()"
                >
                  Delete
                </v-btn>
                <v-btn
                  color="blue darken-1"
                  @click="dialog_delete = false;"
                >
                  Cancel
                </v-btn>
              </v-card-actions>
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
      dialog_delete : false,
      delete_type : null,
      delete_message : null,
      delete_confirm : null,
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
        this.get_current_question_by_id(question_id);
        this.get_current_choice_by_id(choice_id);
        this.dialog = true;
      },

      v_form_question: function(question_id){
        if(question_id=='cl_create_new_question'){
          this.reset_current_question()
        }
        else{
          this.get_current_question_by_id(question_id);
        }
        this.dialog_risk = true;
      },

      v_form_delete: function(question_id, choice_id){
        this.get_current_question_by_id(question_id);        
        if( this.delete_type == 'choice' ){
          this.get_current_choice_by_id(choice_id);
          this.delete_message = "Choice (" + this.current_choice.id + ") \n"+this.current_choice.title;
        }
        if( this.delete_type == 'question'){
          this.delete_message = "Question (" + this.current_question.id + ") \n"+this.current_question.title;
        }
        this.dialog_delete = true;
      },

      update_current_choice(){
        var target_c = this.get_current_choice_index();
        var target_q = this.get_current_question_index();
        var target_p = this.get_current_pillar_index();
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
        var target_q = this.get_current_question_index();
        var target_p = this.get_current_pillar_index();
        var obj;
        try{
          obj = JSON.parse(this.c_riskrule);
          this.current_question.riskRules = obj;
        }catch(err)
        {
          alert("Invalid format for riskRules : " + err.message);
        }
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

      get_current_choice_by_id(cid){
        this.current_question.choices.forEach((c) => {
          if(c.id == cid){
            this.current_choice = c;
          }
        });
      },

      get_current_question_by_id(qid){
        this.questions.forEach((q) => {
          if(q.id == qid){
            this.current_question = q;
          }
        });
        this.c_riskrule = JSON.stringify(this.current_question.riskRules);        
      },

      get_current_choice_index(){
        return this.current_question.choices.indexOf(this.current_choice);
      },

      get_current_question_index(){
        return this.current_pillar.questions.indexOf(this.current_question);
      },

      get_current_pillar_index(){
        return this.custom_lens.pillars.indexOf(this.current_pillar);
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
          id :'',
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
        id: '',
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
      },

      delete_current_object() {
        var target_c = -1;
        var target_q = this.get_current_question_index();
        var target_p = this.get_current_pillar_index();
        if(target_p > -1 && target_q > -1)
        {
          if(this.delete_type=='question'){
            this.custom_lens.pillars[target_p].questions.splice(target_q, 1);
            alert("question deleted:" + target_q);
            this.delete_confirm = null;
            this.v_load_pillar(this.current_pillar.id, this.current_pillar.title);
          }

          if(this.delete_type=='choice'){
            target_c = this.get_current_choice_index();
            if (target_c > -1){
              this.custom_lens.pillars[target_p].questions[target_q].choices.splice(target_c, 1);
              alert("choice deleted:" + target_c);
              this.delete_confirm = null;
            }

          }

        }
      }, 

      load_template() {
          this.reset_current_choice();
          this.reset_current_question();
          this.custom_lens = {"schemaVersion":"2021-11-01","name":"Replace with lens name","description":"Replace with your description","pillars":[{"id":"pillar_id_1","name":"Pillar 1","questions":[{"id":"pillar_1_q1","title":"My first question","description":"Description isn't a necessary property here for a question, but it might help your lens users.","choices":[{"id":"choice1","title":"Best practice #1","helpfulResource":{"displayText":"It's recommended that you include a helpful resource text and URL for each choice for your users.","url":"https://aws.amazon.com"},"improvementPlan":{"displayText":"You must have improvement plans per choice. It's optional whether or not to have a corresponding URL."}},{"id":"choice2","title":"Best practice #2","helpfulResource":{"displayText":"It's recommended that you include a helpful resource text and URL for each choice for your users.","url":"https://aws.amazon.com"},"improvementPlan":{"displayText":"You must have improvement plans per choice. It's optional whether or not to have a corresponding URL."}}],"riskRules":[{"condition":"choice1 && choice2","risk":"NO_RISK"},{"condition":"choice1 && !choice2","risk":"MEDIUM_RISK"},{"condition":"default","risk":"HIGH_RISK"}]}]}]};
          this.custom_lens.ready = true;
          this.v_update_pillars();
      }
    },
    beforeMount(){
    this.reset_current_choice();
    this.reset_current_question();
    }
  }
</script>