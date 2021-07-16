<template>
  <div>
    <div class="menu-btn">
      
      <button
        type="button"
        class="btn btn-default btn-sm move-today"
        @click="moveToday"
      >
        Today
      </button>
      <button
        type="button"
        class="btn btn-default btn-sm move-day"
        @click="movePrev"
      >
        &lt;
      </button>
      <button
        type="button"
        class="btn btn-default btn-sm move-day"
        @click="moveNext"
      >
        &gt;
      </button>

      <select v-model="selected">
      <option
        v-for="(options, index) in memberOptions"
        :value="options.value"
        :key="index"
      >
        {{ options.title }}
      </option>
    </select>

      <span
        class="render-range"
        style="
          font-size: 2rem;
          display: inline-block;
          position: absolute;
          left: 50%;
          width: 20%;
          margin: 0 0 0 -5%;
        "
        >{{ dateRange }}</span
      >
      
    </div>

    <calendar
      style="height: 800px"
      ref="tuiCal"
      :schedules="scheduleList"
      :view="selectedView"
      :theme="theme"
      :month="month"
      :isReadOnly="false"
      :useDetailPopup="useDetailPopup"
      :useCreationPopup="useCreationPopup"
      @beforeDeleteSchedule="onBeforeDeleteSchedule"
      @clickSchedule="onClickSchedule"
    />
  </div>
</template>
<script>
import "tui-calendar/dist/tui-calendar.css";
import { Calendar } from "@toast-ui/vue-calendar";
import axios from "axios";

export default {
  name: "Calendar",
  components: {
    calendar: Calendar,
  },
  data() {
    return {
      memberOptions: [{
        title: 'All',
        value: 'All'
      }],
      selected: 'All',
      scheduleList: [
      ],
      selectedView: "month",
      dateRange: "",
      scheduleView: ["time"],
      theme: {
        "month.dayname.height": "30px",
        "month.dayname.borderLeft": "1px solid #ffffff",
        "month.dayname.textAlign": "center",
        "month.weekend.backgroundColor": "#fafafa",
        "month.holidayExceptThisMonth.color": "#ffffff",
      },
      month: {
        workweek: true,
      },
      isReadOnly: false,
      useDetailPopup: true,
      useCreationPopup: false,
      months: [
        "00",
        "01",
        "02",
        "03",
        "04",
        "05",
        "06",
        "07",
        "08",
        "09",
        "10",
        "11",
        "12",
      ],
      days: [
        "00", "01", "02", "03", "04", "05", "06", "07", "08", "09", "10", "11", "12", "13", "14", "15", 
        "16", "17", "18", "19", "20", "21", "22", "23", "24", "25", "26", "27", "28", "29", "30", "31",
      ],
      types: {SO: '짧은 휴가', LO: '긴 휴가', AH: '오전 반차', PH: '오후 반차', AQ: '오전 반반차', PQ: '오후 반반차',},
      members: [],
      teamId: '',
      empId: '',
      name: '',
    };
  },
  watch: {
    selected() {
      this.loadSchedule();
    }
    
  },
  methods: {
    init() {
      this.setRenderRangeText();
    },
    setRenderRangeText() {
      const { invoke } = this.$refs.tuiCal;
      const view = invoke("getViewName");
      const calDate = invoke("getDate");
      let year = calDate.getFullYear();
      let month = calDate.getMonth() + 1;
      let date = calDate.getDate();
      let dateRangeText = "";
      switch (view) {
        case "month":
          dateRangeText = `${year}.` + this.months[month];
          break;
        default:
          dateRangeText = `${year}-` + this.months[month] + `-${date}`;
      }
      this.dateRange = dateRangeText;
    },
    getColor(n) {
      var colors = [
        "#DF75DB",
        "#9905D8",
        "#EB6464",
        "#FFD732",
        "#FF8200",
        "#A8F552",
        "#00C3FF",
        "#46649B",
        "#787878",
        "#FF9800",
        "#757575",
        "#3F51B5",
      ];
      var len = colors.length;
      return colors[(n - 1)%len];
    },
    movePrev() {
      this.$refs.tuiCal.invoke("prev");
      this.setRenderRangeText();
    },
    moveNext() {
      this.$refs.tuiCal.invoke("next");
      this.setRenderRangeText();
    },
    moveToday() {
      this.$refs.tuiCal.invoke("today");
      this.setRenderRangeText();
    },
    onClickSchedule(e){
      console.log(e);
      if(e.schedule.id !== this.empId){
        hideButtons();
      }
    },
    onBeforeDeleteSchedule(res) {
      console.group("onBeforeDeleteSchedule");
      if(res.schedule.id === this.empId){
          axios
        .delete('https://463d2ocmf7.execute-api.ap-northeast-2.amazonaws.com/vacation/vacation', {
          data: {
            empId: res.schedule.id,
            startDate: res.schedule.start._date.getFullYear()+'-'+this.months[res.schedule.start._date.getMonth()+1]+'-'+this.days[res.schedule.start._date.getDate()],
            endDate: res.schedule.end._date.getFullYear()+'-'+this.months[res.schedule.end._date.getMonth()+1]+'-'+this.days[res.schedule.end._date.getDate()]
          }
        })
        .then((res) => {
          console.log(res);
          this.loadSchedule();
        })
        .catch((err) => {
          console.log(err);
        });
      }else{
        alert('자신의 휴가 일정만 삭제 가능합니다')
      }
      
    },
    getQueryStringObject() {
      var a = window.location.search.substr(1).split('&');
      if (a == "") return {};
      var b = {};
      for (var i = 0; i < a.length; ++i) {
          var p = a[i].split('=', 2);
          if (p.length == 1)
              b[p[0]] = "";
          else
              b[p[0]] = decodeURIComponent(p[1].replace(/\+/g, " "));
      }
      this.teamId = b.teamid
      this.empId = b.empid
      this.name = b.name;
    },
    loadSchedule() {
        axios
        .get('https://463d2ocmf7.execute-api.ap-northeast-2.amazonaws.com/vacation/vacation', {
        params: {
          teamId: decodeURIComponent(this.teamId),
          date: this.dateRange.substring(0,7).replaceAll('.', '-')
        }
        })
        .then((res) => {
          this.scheduleList = [];
          for(var i=0; i<res.data.length; i++){
            if(this.selected === 'All' || this.selected === res.data[i].empid){
              var obj = new Object();
              obj.id = res.data[i].empid,
              obj.calendarId= "1";
              obj.title= res.data[i].name,
              obj.body= '<b>' + this.types[res.data[i].type] + '</b> <br>' + res.data[i].content,
              obj.category= "time",
              obj.dueDateClass= "",
              obj.start= res.data[i].startdate,
              obj.end= res.data[i].enddate,
              obj.borderColor= this.getColor(i+1),
              obj.color= "#ffffff",
              obj.bgColor= this.getColor(i+1),
              this.scheduleList.push(obj);
            }
            
          }
        })
        .catch((err) => {
          console.log(err);
        });
      
    },
    async getAllMember() {
      const memberSet = new Set();
      await axios
        .get('https://463d2ocmf7.execute-api.ap-northeast-2.amazonaws.com/vacation/vacation', {
        params: {
          teamId: decodeURIComponent(this.teamId),
          date: this.dateRange.substring(0,7).replaceAll('.', '-')
        }
        })
        .then((res) => {
          for(var i=0; i<res.data.length; i++){
            if(!memberSet.has(res.data[i].empid)){
              memberSet.add(res.data[i].empid)
              var obj = new Object(); 
              obj.title = res.data[i].name;
              obj.value = res.data[i].empid;
              this.memberOptions.push(obj);
            }
          }
        })
        .catch((err) => {
          console.log(err);
        });
    }
  },
   async mounted() {
    this.init();
    this.getQueryStringObject();
    await this.getAllMember();
    this.loadSchedule();
    
  },
};
function hideButtons(){
    var buttons = document.getElementsByClassName("tui-full-calendar-section-button")[0];
    if(buttons){
        buttons.style.display = "none";
    }
}
</script>
<style>
/* 버튼 수정 */
.btn-default {
  border-radius: 25px;
  border: 1px solid #ccc;
  font-size: 15px;
  padding: 10px 14px;
  text-align: center;
  margin-right: 5px;
  background: white;
}

.menu-btn {
  margin: 50px 0px 30px 0px;
}

.render-range {
  font-size: 18px;
}

/* 캘린더 UI 수정 */
.tui-full-calendar-popup-edit {
    display: none;
}

.tui-full-calendar-popup-delete {
    width: calc(100% - 1px);
}

.tui-full-calendar-popup-vertical-line {
    display: none;
}

.tui-full-calendar-popup-section {
    font-size: 0;
    min-height: 0px;
}

div.tui-full-calendar-popup-section.tui-full-calendar-section-header > div.tui-full-calendar-popup-detail-date.tui-full-calendar-content {
    line-height: 0px;
    height: 0px;
    font-size: 0px !important;
    line-height: 2;
}

div.tui-full-calendar-popup-container > div.tui-full-calendar-section-detail > div {
  border-top: 1px solid #e5e5e5;
  padding-top: 10px;
}

/* select */
select {
  width: 100px; 
  padding: 6px 10px; 
  font-size: 0.9rem;
  font-family: inherit; 
  background-size: 15px 15px;
  border: 1px solid #333;
  background-color: #fff;
  border-radius: 0; 
}

select:focus {
  border-color: #0094e1;
}
</style>
