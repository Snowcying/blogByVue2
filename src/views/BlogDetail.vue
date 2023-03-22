<template>
  <div>
    <Header></Header>

    <el-dialog
        title="警告"
        :visible.sync="dialogVisible"
        width="30%"
        >
      <span>确定删除文章吗？</span>
      <span slot="footer" class="dialog-footer">
    <el-button @click="dialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="handleClose">确 定</el-button>
  </span>
    </el-dialog>

    <div class="mblog" id="mblog">
      <h2> {{ blog.title }}</h2>
      <el-link  icon="el-icon-edit" v-if="ownBlog">
        <router-link :to="{name: 'BlogEdit', params: {blogId: blog.id}}" >
          编辑
        </router-link>
      </el-link>
      <el-link  icon="el-icon-delete" v-if="ownBlog">
        <el-link type="danger" @click="dialogVisible = true">删除</el-link>
      </el-link>
      <el-divider></el-divider>
      <div class="markdown-body" v-html="blog.content"></div>

    </div>

  </div>
</template>

<script>
import 'github-markdown-css'
import Header from "../components/Header";
import Element from "element-ui";

export default {
  name: "BlogDetail.vue",
  components: {Header},
  data() {
    return {
      blog: {
        id: "",
        title: "",
        content: ""
      },
      ownBlog: false,
      dialogVisible: false,
    }
  },
  created() {
    const blogId = this.$route.params.blogId
    console.log(blogId)
    const _this = this
    this.$axios.get('/blog/' + blogId).then(res => {
      const blog = res.data.data
      _this.blog.id = blog.id
      _this.blog.title = blog.title

      let MardownIt = require("markdown-it")
      let md = new MardownIt()

      let result = md.render(blog.content)
      _this.blog.content = result
      _this.ownBlog = (blog.userId === _this.$store.getters.getUser.id)

    })
  },
methods:{
  handleClose() {
    const blogId = this.$route.params.blogId
    const _this = this
    this.$axios.delete('blog/'+blogId,{
      headers: {
        "Authorization": localStorage.getItem("token")
      }
    })
        .then(res => {
          _this.$router.push("/")
          Element.Message.success("删除成功", {duration: 2 * 1000})
          // done();
        })
        .catch(_ => {});
  }
},
}
</script>

<style scoped>
.mblog {
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
  width: 100%;
  min-height: 700px;
  padding: 20px 15px;
}
a{
  text-decoration: none;
  color: #000;
}
#mblog{
  max-width: 1200px;
  margin: 0 auto;
}

</style>