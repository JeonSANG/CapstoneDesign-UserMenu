<!-- 기본 제공 레이아웃 1  persistent-->
<template>
<div class="container" style="width:100%; padding:1%;"> 
<v-container style="color:white" class="text_set"> 
    <!--스낵바 : 경고 창 출력-->
    <v-snackbar 
        v-model="snackbar"
        :timeout="timeout" 
        :top="'top'" 
    >
        {{ snackbar_text }}
        <v-btn 
            @click.native="snackbar = false"
            color="pink" 
            flat icon 
        >
            <v-icon large> close </v-icon>
        </v-btn>
    </v-snackbar>

    <!-- 카테고리 영역 -->
    <v-layout xs12> 
            <v-card style="margin:auto">
            <v-bottom-nav :value="true" :active.sync="e2">
                <v-btn 
                    v-for="(value,key) in categorys" 
                    :key="key" 
                    :value="value" 
                    @click="click_category"
                >
                    {{value}} 
                </v-btn>
            </v-bottom-nav>
        </v-card>
    </v-layout> 
    
    <!-- 메뉴 영역 -->
    <div 
        v-for="n in range(0, menu_num-1)" 
        :key="n" 
        class="menu_body"
    >        
        <v-card color="white" style="color: black;">                
            <!-- 메뉴 이미지 -->
            <div class="tem_img">
                <!-- <img src="./example7.jpg" class="inner_img">  -->
                <img :src="get_menus[n].path + get_menus[n].filename" class="inner_img">
            </div>

            <!-- 메뉴 정보 -->
            <v-card-title primary-title style="padding:2%;">                    
                <!-- 메뉴 명 -->
                <div class="menu-outer" style="width: 100%;">
                    <h3 class="menu_info"> 
                        {{get_menus[n].name}}
                    </h3>
                </div>                 

                <!-- 메뉴 설명 -->
                <div class="menu_expl">
                    <div class="menu_info"> 
                        {{get_menus[n].explanation}}
                    </div>
                </div>
                            
                <!-- 메뉴 점심 메뉴 / 저녁 메뉴 구분 -->
                <div class="menu-outer" style="width: 100%;">
                    <h4 class="menu_info"> 
                        Time : &nbsp;{{get_menus[n].remark}} 
                    </h4>
                </div> 
                                
                <!-- 메뉴 가격 -->
                <div class="menu-outer" style="width:50%;">
                    <b class="menu_info" style="margin-top:2%; margin-left:2%;"> 
                        {{get_menus[n].price}}&nbsp;円
                    </b>
                </div> 
                                                                                
                <!-- 주문하기 버튼 -->
                <div class="menu-outer" style="width: 50%;">      
                    <button 
                        :id="n" 
                        @click="select_menu" 
                        style= "width:100%; 
                                padding-top:5%; 
                                color:cadetblue;"
                    >
                        {{selectBtn}} 
                    </button>
                </div>
            </v-card-title>
        </v-card>
    </div> 
</v-container> 
</div>
</template>
        
<script type="text/javascript"> 
import VueAxios from 'vue-axios';
import axios from 'axios';
import { EventBus } from './eventBus.js';
  
export default {

    created() {
        let url           = '/menu/getCategory';
        let get_categorys = null;
        let shop_id       = this.$route.params.shop_id;      // 샵 아이디 
        let notion = this.$session.get('user_country');
 
        // 카테고리 요청하기.
        axios.post(url, {
          'shop_id' : shop_id
        })
        .then( (response) => {
            get_categorys = response.data.category;
            this.categorys = this.unique(get_categorys); // 카테고리 중복 값 제거.
        })
        .catch((ex)=>{　
        });          　        

        switch(notion){
            case 'Korea': this.selectBtn = '선택하기';   break;
            case 'Japan': this.selectBtn = '選択';      break;
            case 'China': this.selectBtn = '選択';      break;
            case 'USA':   this.selectBtn = 'Select';   break;
        }  
    },
    
    data() {
        return {  
            // 스낵바 설정값
            timeout         : 2000,
            snackbar_text   : '',
            snackbar        : false,

            e2              : null,     // 카테고리 클릭 값
            categorys       : null,     // 카테고리 배열
            get_menus       : null,     // 해당 카테고리 메뉴들
            menu_num        : 0,        // 메뉴 출력 v-layout 갯수 
            select_menus    : [],       // 상위 컴퍼넌트에 보낼 값
            shop_id         : null,     // 가게 아이디 값 

            selectBtn    : '選択'
        }
    },

    methods : {
        // 메뉴 카테고리 클릭
        click_category : function() {
            let category = event.target;                    // 선택한 카테고리 
            let url      = "";                              // 서버에 요청할 주소 
            let shop_id  = this.$route.params.shop_id;      // 샵 아이디 
             
            // 클릭한 값 검사
            if(category.value === undefined) {
                category = category.parentNode.value;
            } else {
                category = category.value;
            }
            url = '/menu/getMenu/' + shop_id + '/' + category;
 
            // 클릭한 카테고리의 메뉴 호출
            axios.get(url)
            .then( (response) => {
                this.get_menus = response.data.menu; 
                this.menu_num  = this.get_menus.length;                  
            })
            .catch((ex)=>{　
            });
                
            this.translate();   
        }, // end of click_category

        // 칼럼 번역하기.
        translate : function( ){
            let notion = this.$session.get('user_country');
            
            for(let i=0; i < this.menu_num; i++){

                switch(this.get_menus[i].remark){
                    case '디너 메뉴' : 
                             if(notion == 'Korea') this.get_menus[i].remark = '디너 메뉴'
                        else if(notion == 'Japan') this.get_menus[i].remark = 'ディナ'
                        else if(notion == 'China') this.get_menus[i].remark = 'Dinner '
                        else if(notion == 'USA')   this.get_menus[i].remark = 'Dinner '                        
                    break;

                    case '런치 메뉴' :
                             if(notion == 'Korea') this.get_menus[i].remark = '런치 메뉴'
                        else if(notion == 'Japan') this.get_menus[i].remark = 'ランチ'
                        else if(notion == 'China') this.get_menus[i].remark = 'Lunch'
                        else if(notion == 'USA')   this.get_menus[i].remark = 'Lunch'  
                    
                    break;

                    case '상관 없음' : 
                             if(notion == 'Korea') this.get_menus[i].remark = '상관 없음'
                        else if(notion == 'Japan') this.get_menus[i].remark = '関係ない'
                        else if(notion == 'China') this.get_menus[i].remark = 'All time'
                        else if(notion == 'USA')   this.get_menus[i].remark = 'All time'                      
                    break;
                }                
            }
        },

        // 배열 중복 값 제거, 인자는 서버에 받은 카테고리 목록
        unique : function(argArr) {
            var array = [];
 
            for(let i=0; i < argArr.length; i++){
                // array에 없는 값이면 push
                if( !array.includes(argArr[i]['category']) ) { 
                    array.push(argArr[i]['category']);
                }
            }
            return array;
        },

        // v-for 용 함수, start에서 시작해서 end까지 1식 반환
        range : function (start, end) {
            return Array(end - start + 1).fill().map((_, idx) => start + idx);
        },

        // 주문하기 클릭한 메뉴
        select_menu : function () {
            let menu = event.target;                            // 클릭한 메뉴 가져오기. 
            let arr  = [];

            arr['menu'] = this.get_menus[menu.id]
            this.select_menus.push(arr);    // 보낼 값 배열에 담기.  
            EventBus.$emit('select_menus', this.select_menus);
            this.snackbar_text = '선택하신 메뉴가 추가 되었습니다.';
            this.snackbar = true;
        }
    }// end of method

}// end of export default
</script> 

<style scoped>
/* 모바일 */
@media (max-width: 639px){  
    .text_set  { font-size: 15px; } 
    .menu_body { width:100%; margin:1%;} 
    .menu_expl { padding-bottom: 15%; }
}

/* 테블릿 */
@media (min-width: 640px) and (max-width: 1299px){  
    .text_set   { font-size: 13px; } 
    .menu_body  { width:47%; margin:1%; }
    .menu_expl  { padding-bottom: 15%; }
}

/* 데스트 탑 */
@media (min-width: 1300px){  
    .text_set   { font-size: 12px; } 
    .menu_body  { width:31%; margin :1%;}
    .menu_expl  { padding-bottom: 25%; }
} 

/* 메뉴 */
.menu_body { 
    float: left;
}

/* 메뉴 이미지 바깥 */
.tem_img {
    width: 100%;
    height: 0;
    padding-bottom: 70%;
    overflow: hidden;
    position: relative;
}
/* 메뉴 이미지 안쪽 */
.inner_img { 
    top: 0;
    left: 0;
    width: 105%;
    height: 105%; 
    position: absolute;
    object-fit: cover;
} 

/* 메뉴 정보 길고 얇은거 */
.menu-outer { 
    height: 0;  
    padding-bottom: 11%;   
    overflow: hidden;
    position: relative;
}
/* 메뉴 설명 */
.menu_expl { 
    width: 100%;
    height: 0;     
    overflow: auto;
    position: relative;
}

.menu_info {
    top: 0;
    left: 0;
    width: 100%;
    height: 100%; 
    position: absolute;
}  
</style>
