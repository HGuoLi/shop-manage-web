<template>
  <v-form v-model="valid" ref="BrandForm">
    <v-text-field
      label="品牌名称"
      v-model="brand.name"
      :rules="[v => !!v || '品牌名称不能为空']"
      :counter="10"
      required
    />
    <!--v => v.length <=10 || '品牌名称不能超过10个字'-->
    <v-text-field
      label="品牌首字母"
      v-model="brand.letter"
      :rules="[v => !!v || '品牌首字母不能为空']"
      required
      mask="A"
    />
    <v-cascader url="/item/category/list" required
                v-model="brand.categories"
                multiple label="请选择商品分类"/>
    <v-layout row>
      <v-flex xs3>
        <span style="font-size: 19px; color: #444">品牌LOGO：<span style="font-size: 16px; color:red">(图片大小不能大于5MB)</span></span>
      </v-flex>
      <v-flex>
        <v-upload
          v-model="brand.image" url="/upload/image" :multiple="false" :pic-width="250" :pic-height="90"
        />
      </v-flex>
    </v-layout>
    <v-layout class="my-4">
      <v-spacer/>
      <v-btn @click="submit" color="primary">提交</v-btn>
      <v-btn @click="clear" color="warning">重置</v-btn>
    </v-layout>
  </v-form>
</template>

<script>
  import config from '@/config';

  export default {
    name: "brand-form",
    props: {
      oldBrand: Object,
      isEdit: {
        type: Boolean,
        default: false
      },
      show: {
        type: Boolean,
        default: true
      }
    },
    data() {
      return {
        baseUrl: config.api,
        valid: false,
        brand: {
          name: "",
          image: "",
          letter: "",
          categories: []
        },
        imageDialogVisible: false
      }
    },
    watch: {
      oldBrand: {
        deep: true,
        handler(val) {
          Object.deepCopy(val, this.brand);
        }
      }
    },
    methods: {
      submit() {
        // 1、表单校验
        if (this.$refs.BrandForm.validate()) {
          // 2、定义一个请求参数对象，通过解构表达式来获取brand中的属性
          const {categories, ...params} = this.brand;
          // 3、数据库中只要保存分类的id即可，因此我们对categories的值进行处理,只保留id，并转为字符串
          params.cids = categories.map(c => c.id).join(",");
          // 4、将数据提交到后台
          this.$http.post('/item/brand', this.$qs.stringify(params))
            .then(() => {
              //5、关闭窗口
              this.$emit("close");
              this.clear();
              //6、弹出提示
              this.$message.success("保存成功！");
            })
            .catch(() => {
              this.$message.error("保存失败！");
            });
        }
      },
      clear() {
        // 重置表单
        this.$refs.BrandForm.reset();
        this.categories=[];
      },
      closeWindow() {
        this.$emit("close");
      }
    }
  }
</script>

<style scoped>

</style>
