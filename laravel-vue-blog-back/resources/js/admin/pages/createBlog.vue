<template>
    <div>
			<div class="container-fluid">

				<!--~~~~~~~ TABLE ONE ~~~~~~~~~-->
				<div class="_1adminOverveiw_table_recent _box_shadow _border_radious _mar_b30 _p20">
					<p class="_title0">Create a New Blog </p>
                      <div class="_input_field">
                         <Input type="text" v-model="data.title" placeholder="Title"/>
                       </div>
                       <div class="space"></div>
					<div class="_overflow _table_div _blog_editor">
					   <editor
                         
                         ref="editor"
                         autofocus
                         holder-id="codex-editor"
                         save-button-id="save-button"
                         :init-data="initData"
                          @save="onSave"
                          :config="config" 
                        
                       />
                       	</div>

                        <div class="_input_field">
                         <Input type="textarea" v-model="data.post_except" rows="4" placeholder="Enter Post Except.."/>
                       </div>
                       <div class="_input_field">
                       <Select filterable multiple placeholder="Select Tag" v-model="data.tag_id">
                          <Option v-for="(t,i) in tag" :value="t.id" :key="i">{{t.tagName}}</Option>
                       </Select>
                       </div>
                        <div class="_input_field">
                       <Select filterable multiple placeholder="select Category" v-model="data.category_id">
                          <Option v-for="(c,i) in category" :value="c.id" :key="i">{{c.categoryName}}</Option>
                       </Select>
                       </div>
                          <div class="_input_field">
                         <Input type="textarea" rows="4" v-model="data.metaDescription" placeholder="Enter Meta Description.."/>
                       </div>

                       
                      
				<div class="_input_field">
                          <Button @click="save" :loading="isCreating" :disable="isCreating">{{isCreating ? 'Please Wait..' : 'Create Blog'}}</Button>
                       </div>
				</div>

			</div>
		</div>
</template>

<script>
//import ImageTool from '@editorjs/image';
export default{
       data(){
         return {
            config: {
              
                },
             initData:null,
                data:{
                   title:'',
                   post:'',
                   post_except:'',
                   metaDescription:'',
                   category_id:[],
                   tag_id:[],
                   jsonData:null,
                },
                articleHTML: '',
                category: [],
                tag: [],
                isCreating:false,
         }
      },
  
   


    methods:{
        
        async onSave(response){
            var data = response
           await this.outputHtml(data.blocks)
           this.data.post = this.articleHTML
           this.data.jsonData = JSON.stringify(data)
           if(this.data.post.trim() == '') return this.e('Post is required')
           if(this.data.title.trim() == '') return this.e('Title is required')
           if(this.data.post_except.trim() == '') return this.e('Post except is required')
           if(!this.data.tag_id.length) return this.e('Tag  is required')
           if(!this.data.category_id.length) return this.e('Category is required')
           if(this.data.metaDescription.trim() == '') return this.e('Meta Description is required')

           this.isCreating = true
           const res = await this.callApi('post','app/create_blog',this.data)
           if(res.status == 200){
               this.s('Blog Has Been created Succesfully')
               this.$router.push('/blogs') //redirect
           }else{
            if(res.status == 422){
               for(let i in res.data.errors){
                   this.e(res.data.errors[i][0])
               }
           }else{
               this.swr();
           }   
           }
           this.isCreating = false
          
        },
          async save(){
             this.$refs.editor.save()
        },
        outputHtml(articleObj){
              articleObj.map(obj => {
                  switch(obj.type){
                      case 'paragraph':
                      this.articleHTML += this.makeParagraph(obj);
                      break;
                      case 'image':
                      this.articleHTML += this.makeImage(obj);
                      break;
                      case 'header':
                      this.articleHTML += this.makeHeader(obj);
                      break;
                      case 'raw':
                      this.articleHTML += `<div class="ce-block">
                        <div class="ce_block_content">
                        <div class="ce-code">
                        <code>${obj.data.html}</code>
                        </div>
                        </div>
                        </div>\n`;
                      break;
                      case 'code':
                      this.articleHTML += this.makeCode(obj);
                      break;
                      case 'list':
                      this.articleHTML += this.makeList(obj);
                      break;
                      case 'quote':
                      this.articleHTML += this.makeQuote(obj);
                      break;
                      case 'warning':
                      this.articleHTML += this.makeWarning(obj);
                      break;
                      case 'checklist':
                      this.articleHTML += this.makeChecklist(obj);
                      break;
                       case 'embed':
                      this.articleHTML += this.makeEmbed(obj);
                      break;
                       case 'delimeter':
                      this.articleHTML += this.makeDelimeter(obj);
                      break;
                      default:
                      return '';
                     
                  }
              });
        },

   },
   
        async created(){
            const [cat,tag] = await Promise.all([
               this.callApi('get','app/get_categories'),
               this.callApi('get','app/get_tags'),
            ]);
            if([cat,tag].status = 200){
                this.category = cat.data
                this.tag = tag.data
            }else{
                this.swr()
            }
        }

   
}



</script>

<style>
._blog_editor{
    width:717px;
    margin-left:160px;
    padding:4px 7px;
    font-size:14px;
    border:1px solid #dcdee2;
    border-radius:4px;
    color:#515a6e;
    background-color:#fff;
    background-image:none;
    z-index:-1;
}
._blog_editor:hover{
    border:1px solid #57a3f3;
}
._input_field{
    margin:20px 0 0 160px;
    width:717px;
}
</style>
