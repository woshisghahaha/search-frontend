<template>
  <div class="home">
    <!-- 搜索栏 -->
    <!-- <a-input-search
      v-model:value="searchText"
      placeholder="input search text"
      enter-button="Search"
      size="large"
      @search="onSearch"
    /> -->
    <a-auto-complete
      v-model:value="searchText"
      :options="options"
      style="width: 85%"
      placeholder="input search text"
      @select="onSearch"
      @search="getSearchPrompt"
    />
    <a-button
      type="primary"
      style="left-margin: 10px; width: 15%"
      @click="onSearch(searchText)"
      >Search
    </a-button>
    <!-- 搜索栏下的标签选项 -->
    <!-- 把myAxios从服务器得到的帖子列表展示一下 -->
    <!-- {{JSON.stringify(postList)}} -->
    <MyDivider />
    <a-tabs v-model:activeKey="activeKey" @change="onTabChange">
      <a-tab-pane key="post" tab="文章">
        <!-- 把myAxios拿到的数据传给postList组件,让他展示帖子列表 -->
        <PostList :post-list="postList" />
      </a-tab-pane>
      <a-tab-pane key="picture" tab="图片">
        <PictureList :pic-list="picList" />
      </a-tab-pane>
      <a-tab-pane key="video" tab="视频">
        <VideoList :video-list="videoList" />
      </a-tab-pane>
      <a-tab-pane key="user" tab="用户">
        <UserList :user-list="userList" />
      </a-tab-pane>
      <!--
        // 此功能由于需要ES，而服务器带不动，暂时取消
        <a-tab-pane key="hotel" tab="酒店">
        <HotelList :hotel-list="hotelList" />
      </a-tab-pane>
      -->
    </a-tabs>
  </div>
  <a-pagination
    show-size-changer
    :page-size-options="pageSizeOptions"
    v-model:current="current"
    v-model:pageSize="pageSizeParam"
    :total="total"
  />
</template>

<script setup lang="ts">
import { ref, watchEffect } from "vue";
import PostList from "@/components/PostList.vue";
import PictureList from "@/components/PictureList.vue";
import UserList from "@/components/UserList.vue";
import MyDivider from "@/components/MyDivider.vue";
import VideoList from "@/components/VideoList.vue";
import { useRoute, useRouter } from "vue-router";
import myAxios from "@/plugins/myAxios";

const postList = ref([]);
const userList = ref([]);
const picList = ref([]);
const hotelList = ref([]);
const videoList = ref([]);

//总条数和每页多少条数据
const total = ref(500);
const pageSizeOptions = ref<string[]>(["10", "20", "30", "40", "50"]);

//搜索建议的选择
const options = ref<any[]>([]);

const router = useRouter(); //拿到当前url
//比如/pic/text=aaa，我们要拿的就是aaa
const route = useRoute(); //拿到当前路由参数，或者说查询条件？
let activeKey = route.params.category; //动态路由同步回标签
//设置标签默认值为动态路由参数，这样改变标签，默认标签也会跟着改变
//而不是默认post，否则每次刷新都会跳回默认post

const initSearchParams = {
  type: activeKey,
  text: "",
  pageSize: 5,
  pageNum: ref(1), //响应式变量
};

const searchText = ref(route.query.text || "");

/**
 * 加载单类数据
 */
const loadData = (params: any) => {
  const { type } = params;
  const query = {
    ...params,
    searchText: params.text, //后端是searchText，前端是text，需要转换一下
  };
  if (!type) {
    //type为空可以加载聚合数据
    myAxios.post("/search/all", query).then((res: any) => {
      postList.value = res.postList;
      userList.value = res.userList;
      picList.value = res.pictureList; //左边写前端，右边写后端
      total.value = parseInt(res.total);
    });
  } else {
    myAxios.post("search/all", query).then((res: any) => {
      if (type === "post") {
        postList.value = res.dataList;
        total.value = parseInt(res.total);
      } else if (type === "user") {
        userList.value = res.dataList;
        total.value = parseInt(res.total);
      } else if (type === "hotel") {
        hotelList.value = res.dataList; //左边写前端，右边写后端
        total.value = parseInt(res.total);
      } else if (type === "picture") {
        picList.value = res.dataList; //左边写前端，右边写后端
        total.value = 500;
      } else if (type === "video") {
        videoList.value = res.dataList; //左边写前端，右边写后端
        total.value = 500;
      }
      console.log(res);
      console.log(total.value);
    });
  }
};
/**
 * 如果后端不做聚合搜索，那前端将会需要根据后端的接口和后端的参数进行手动转换
 * 不利于前后端沟通联调
 */
const loadDataOld = (params: any) => {
  const postQuery = {
    ...params,
    searchText: params.text, //后端是searchText，前端是text，需要转换一下
  };
  // 从后端拿帖子列表数据
  // post哪怕是空也要发个空请求体requestBody
  // 后续需要修改res的类型为正确的类型，此处只是为了先跑通
  myAxios.post("post/list/page/vo", postQuery).then((res: any) => {
    postList.value = res.records;
  });
  const userQuery = {
    ...params,
    userName: params.text, //后端是userName，前端是text，需要转换一下
  };
  // 从后端拿用户列表数据
  myAxios.post("user/list/page/vo", userQuery).then((res: any) => {
    userList.value = res.records;
  });
  const picQuery = {
    ...params,
    searchText: params.text, //后端是userName，前端是text，需要转换一下
  };
  // 从后端拿图片列表数据
  myAxios.post("picture/list/page/vo", picQuery).then((res: any) => {
    picList.value = res.records;
  });
};

const searchParams = ref(initSearchParams);
const pageSizeParam = ref(5);
const current = ref(1);

const onPageChange = (page: number) => {
  current.value = page;
};

/*
watchEffect会监控该函数里的变量变化，当参数变化执行我们的函数
目的，把url地址同步到页面状态
这样当我们在某标签搜索刷新页面的时候，就不会跳回默认标签页面
如/pic/text=aaa->刷新，不会变成/user/text=aaa
并且，就算用户没点击搜索，只是在搜索框输入，url也会随之改变
*/
watchEffect(() => {
  searchParams.value = {
    ...initSearchParams, //给个默认值兜底
    text: route.query.text, //把查询信息aaa同步回SearchParams的text属性
    type: route.params.category, //把动态路由的标签同步回type
    pageNum: current.value,
    pageSize: pageSizeParam,
  } as any;
  console.log("标签：", route.params.category);
  console.log("搜索框", route.query.text);
  loadData(searchParams.value); //searchParams改变，重新从后端拿数据
});

/* 比如输入aaa并点击搜索后，url由/pic->/pic/text=aaa */
const onSearch = (value: string) => {
  /* 当用户点击搜索的时候就会在当前url后面加上text=value的查询条件 */
  router.push({
    query: {
      ...searchParams.value,
      text: value,
    },
  });
  current.value = 1;
  //alert(value);
};

/*
标签（文章，图片，用户）改变时， URL也要改变成新标签+搜索栏内容
比如/pic/text=aaa->/user/text=aaa
*/
const onTabChange = (key: string) => {
  //alert(key);
  searchParams.value.type = key;
  router.push({
    path: `/${key}`, //直接指定当前url路径后面跟标签的名字key
    query: searchParams.value, //再跟上用户搜索栏的text
  });
  current.value = 1;
};

const getSearchPrompt = (value: string) => {
  options.value = [];
  console.log(value);
  if (value) {
    const query = {
      params: {
        searchText: value,
        type: searchParams.value.type,
      },
    };
    myAxios.get("search/getSearchPrompt", query).then((res: any) => {
      for (let i = 0; i < res.length; i++) {
        const tempMap = {
          value: res[i],
          color: "red",
        };
        options.value.push(tempMap);
      }
    });
  }
};
</script>
