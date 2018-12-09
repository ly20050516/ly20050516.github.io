<template>
  <div>

    <section class="page-header">

      <div style="position:relative; top:20px;  z-index:2;margin: auto;margin-top:0px;width:48rem;">
        <el-card shadow="never" :body-style="{ padding: '0px'}" style="background-color: transparent">

          <el-row>
            <el-col :span="8" style="text-align: center;padding: 12px 0px 0px 10px;color: #ffffff">
              <el-row>
                <el-col :span="4">
                  <el-popover placement="top" trigger="hover">
                    <div style="text-align: center">
                      <el-progress color="#67C23A" type="circle" :percentage="music.volume"></el-progress>
                      <br>
                      <el-button @click="changeVolume(-10)" icon="el-icon-minus" circle></el-button>
                      <el-button @click="changeVolume(10)" icon="el-icon-plus" circle></el-button>
                    </div>

                    <el-button @click="play" id="play" slot="reference"
                               :icon="music.isPlay?'el-icon-refresh':'el-icon-caret-right'" circle></el-button>
                  </el-popover>
                </el-col>
                <el-col :span="14" style="padding-left: 20px">
                  <el-slider @change="changeTime" :format-tooltip="$util.formatTime" :max="music.maxTime"
                             v-model="music.currentTime" style="width: 100%;"></el-slider>
                </el-col>
                <el-col :span="6" style="padding: 9px 0px 0px 10px;color:#ffffff;font-size: 13px">
                  {{$util.formatTime(music.currentTime)}}/{{$util.formatTime(music.maxTime)}}
                </el-col>
              </el-row>

              <audio ref="music" loop autoplay v-if="audioAutoPlay">
                <source :src="audioUrl" type="audio/mpeg">
              </audio>
              <audio ref="music" loop v-else>
                <source :src="audioUrl" type="audio/mpeg">
              </audio>
            </el-col>

            <el-col :span="14" style="text-align: right;">
              <div style="font-size: 20px;color:#ffffff;margin-top: 5px">
                <b>{{githubUsername}}</b>
              </div>
              <div style="color:#ffffff;">
                <i class="el-icon-location"></i>&nbsp;{{location?location:'未填写地址'}}
                <br>
              </div>
            </el-col>

            <el-col :span="2" style="text-align: center;">
              <img v-popover:bigAvatar :src="avatarUrl"
                   style="margin-top: 4px;margin-right: 10px;width:52px; height:52px; border-radius:5px; border: 1px solid #EBEEF5"
              />
              <el-popover ref="bigAvatar" placement="top-start" :title="githubUsername" width="200" trigger="hover">
                <i class="el-icon-star-on"></i>&emsp;{{name}}
                <br>
                <i class="el-icon-location"></i>&emsp;{{location}}
                <br>
                <img :src="avatarUrl" style="width: 200px;height: 200px;">
              </el-popover>
            </el-col>

          </el-row>

        </el-card>
      </div>

      <div style="position:absolute; top:20px; right:20px; z-index:2;">
        <el-tooltip effect="dark" :content="fullButton.full?'退出':'全屏'" placement="bottom-end">
          <el-button @click="full" :icon="fullButton.full?'el-icon-close':'el-icon-rank'" circle></el-button>
        </el-tooltip>
      </div>

      <h1 class="project-name">{{blogTitle}}</h1>
      <h2 class="project-tagline">{{blogDescribe}}</h2>
      <a :href="'https://github.com/'+githubUsername" class="btn" target="_blank">我的github</a>
      <a href="https://www.jianshu.com/u/7398ccc6f6e2" class="btn" target="_blank">我的简书</a>
    </section>

    <section class="main-content">
      <el-row>
        <el-col :span="4" style="padding-right:10px">
          <sidebar></sidebar>
        </el-col>
        <el-col :span="20" style="padding-left:10px">
          <app-main></app-main>
        </el-col>
      </el-row>

    </section>
    <section class="foot">
      <foot></foot>
    </section>


  </div>
</template>
<script>

  import {mapGetters} from "vuex";
  import Sidebar from "./components/Sidebar";
  import AppMain from "./components/AppMain";
  import Foot from "./components/Foot";

  export default {
    components: {
      Sidebar,
      AppMain,
      Foot,
    },
    data() {
      return {
        music: {
          isPlay: false,
          currentTime: 0,
          maxTime: 0,
          volume: 100
        },
        fullButton: {
          full: false
        },
        topbar: {
          active: ""
        },
        randomIcon: []
      };
    },
    computed: {
      ...mapGetters([
        "githubUsername",
        "blogTitle",
        "blogDescribe",
        "avatarUrl",
        "name",
        "location",
        "blog",
        "fontColor",
        "useBackgroundImage",
        "backgroundColorLeft",
        "backgroundColorRight",
        "audioUrl",
        "mini",
        "followersTotal",
        "followingTotal",
        "audioAutoPlay",
        "webSites"
      ])
    },
    watch: {
      "$refs.music.currentTime": function () {
        console.log(this.$refs.music.currentTime);
      }
    },
    mounted() {
      this.$nextTick(() => {
        setInterval(this.listenMusic, 1000);
      });
      let width = window.innerWidth;
      for (let i = 0; i < 12; i++) {
        let temp = {};
        let left = this.$util.randomInt(10, width - 310);
        if (left > width / 2 - 150) {
          left += 300;
        }
        temp["left"] = left;
        temp["top"] = this.$util.randomInt(20, 300);
        temp["size"] = this.$util.randomInt(20, 40);
        this.randomIcon.push(temp);
      }
    },
    created() {
    },
    methods: {
      selectTopbar(index) {
        //取消菜单选中状态
        this.topbar.active = this.topbar.active == "" ? " " : "";
        switch (index) {
          case "#githubHome":
            window.open("https://github.com/" + this.githubUsername);
            break;
          case "#blog":
            if (this.blog) {
              window.open(
                (this.blog.match(/https?:\/\//i) ? "" : "https://") + this.blog
              );
            } else {
              this.$message({
                message: "博主没有其他博客",
                type: "info"
              });
            }
            break;
          default:
            if (/#webSites-\d+/.test(index)) {
              let i = parseInt(index.split("-")[1]);
              let url = this.webSites[i].url;
              window.open((url.match(/https?:\/\//i) ? "" : "https://") + url);
            }
            break;
        }
      },
      moveIcon(index) {
        let width = window.innerWidth;
        this.randomIcon[index]["top"] = this.$util.randomInt(20, 300);
        let left = this.$util.randomInt(10, width - 310);
        if (left > width / 2 - 150) {
          left += 300;
        }
        this.randomIcon[index]["left"] = left;
      },
      full() {
        if (!this.fullButton.full) {
          this.$util.fullScreen();
          this.fullButton.full = true;
        } else {
          this.$util.fullExit();
          this.fullButton.full = false;
        }
      },
      listenMusic() {
        if (!this.$refs.music) {
          return;
        }
        if (this.$refs.music.readyState) {
          this.music.maxTime = this.$refs.music.duration;
        }
        this.music.isPlay = !this.$refs.music.paused;
        this.music.currentTime = this.$refs.music.currentTime;
      },
      play() {
        if (this.$refs.music.paused) {
          this.$refs.music.play();
        } else {
          this.$refs.music.pause();
        }
        this.music.isPlay = !this.$refs.music.paused;
        this.$nextTick(() => {
          document.getElementById("play").blur();
        });
      },
      changeTime(time) {
        this.$refs.music.currentTime = time;
      },
      changeVolume(v) {
        this.music.volume += v;
        if (this.music.volume > 100) {
          this.music.volume = 100;
        }
        if (this.music.volume < 0) {
          this.music.volume = 0;
        }
        this.$refs.music.volume = this.music.volume / 100;
      }
    }
  };
</script>

<style>
  .page-header {
    color: #fff;
    text-align: center;
    background-size: 100% 100%;
    background-repeat: no-repeat;
    transition: all .6s ease;
  }

  .project-name {
    font-size: 3.25rem;
    margin-top: 20px;
    margin-bottom: 0.1rem;
  }

  .project-tagline {
    font-size: 1.25rem;
    margin-bottom: 2rem;
    font-weight: normal;
    opacity: 0.7;
  }

  .btn:hover {
    color: rgba(255, 255, 255, 0.8);
    text-decoration: none;
    background-color: rgba(255, 255, 255, 0.2);
    border-color: rgba(255, 255, 255, 0.3);
  }

  a:hover {
    text-decoration: underline;
  }

  a:active,
  a:hover {
    outline: 0;
  }

  .btn {
    padding: 0.75rem 1rem;
    display: inline-block;
    margin-bottom: 1rem;
    color: rgba(255, 255, 255, 0.7);
    background-color: rgba(255, 255, 255, 0.08);
    border-color: rgba(255, 255, 255, 0.2);
    border-style: solid;
    border-width: 1px;
    border-radius: 0.3rem;
    transition: color 0.2s, background-color 0.2s, border-color 0.2s;
  }

  a {
    color: #1e6bb8;
    text-decoration: none;
  }

  .btn + .btn {
    margin-left: 1rem;
  }

  .main-content {
    max-width: 110rem;
    padding: 30px;
    margin: 0 auto;
    font-size: 1.1rem;
    word-wrap: break-word;
    background-color: #000000;
  }

  .foot {
    max-width: 67rem;
    margin: 0 auto;
    font-size: 12px !important;
    color: #586069 !important;
    word-wrap: break-word;
  }
</style>
