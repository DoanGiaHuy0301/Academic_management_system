<template>
  <div>
    <h2 class="score">Thông tin điểm số</h2>
    <div v-for="(semester, semesterIndex) in semesters" :key="semesterIndex">
      <div class="semester">{{ `${semester[1]}-${semester[2]}` }}</div>
      <table class="score-table">
        <thead>
          <tr>
            <th>STT</th>
            <th>Tên môn học</th>
            <th>Số tín chỉ</th>
            <th>Quá trình</th>
            <th>Giữa kì</th>
            <th>Cuối kì</th>
            <th>TK</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(score, scoreIndex) in scoreLists[semesterIndex]" :key="scoreIndex">
            <td>{{ scoreIndex + 1 }}</td>
            <td>{{ score.subjectName }}</td>
            <td>{{ score.credit }}</td>
            <td>
              <span v-if="score.scoreDto && score.scoreDto.length > 0">
                <span v-if="score.scoreDto[0].scoreColumnName === 'Quá trình'">
                  {{ score.scoreDto[0].scoreValue || "-" }}
                </span>
              </span>
            </td>
            <td>
              <span v-if="score.scoreDto && score.scoreDto.length > 1">
                <span v-if="score.scoreDto[1].scoreColumnName === 'Giữa kì'">
                  {{ score.scoreDto[1].scoreValue || "-" }}
                </span>
              </span>
            </td>
            <td>
              <span v-if="score.scoreDto && score.scoreDto.length > 2">
                <span v-if="score.scoreDto[2].scoreColumnName === 'Cuối kì'">
                  {{ score.scoreDto[2].scoreValue || "-" }}
                </span>
              </span>
            </td>
            <td></td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
import { ref, onMounted } from "vue";
import { authApi, endpoints } from '@/configs/Apis';
import { mapGetters } from 'vuex';

export default {
     computed: {
    ...mapGetters(["isAuth", "getUser"]),
  },
  data() {
    return {
      semesters: [],
      scoreLists: [],
      err: "",
    };
  },
  created() {
    this.fetchData();
  },
  methods: {
    async fetchData() {
         try {
        const studentUsername = this.getUser.username;
        const response = await authApi().get(
          endpoints["get-student-by-username"].replace("{username}", studentUsername)
        );
        console.log("get-student-by-username",response.data)
        const studentId = response.data.id;
        const semesterResponse = await authApi().get(
          endpoints["semester-student"] + `?studentId=${studentId}`
        );

        // Lưu thông tin về semesters vào state
        this.semesters = semesterResponse.data;
        console.log(this.semesters);

        const scoreListsValue = [];

        for (const semester of semesterResponse.data) {
          const semesterId = semester[0];

          const scoreEndpoint =
            endpoints["score-list"] +
            `?studentId=${studentId}&semesterId=${semesterId}`;
          const scoreResponse = await authApi().get(scoreEndpoint);

          // Lưu thông tin về điểm số của từng học kỳ vào mảng scoreLists
          scoreListsValue.push(scoreResponse.data);
        }

        // Lưu mảng scoreLists vào state
        this.scoreLists = scoreListsValue;
      } catch (err) {
        err.value = true;
      }
    }
  },
  setup() {
   
  },
};
</script>

<style scoped>
/* Thêm các lớp CSS cho phù hợp với giao diện của bạn tại đây */
</style>
