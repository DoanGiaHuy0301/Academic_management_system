<template>
  <div class="container" style="margin-bottom: 200px">
    <div class="row">
      <form @submit.prevent="handleSubmit">
        <div class="col-12 d-flex studentScore">
          <div class="input-studentScore">
            <label for="subjectSelect">Chọn môn học:</label>
            <select
              class="form-control"
              id="subjectSelect"
              v-model="selectedSubject"
              @change="handleSubjectChange"
            >
              <option value="">Chọn môn học</option>
              <option
                v-for="(subject, index) in subjectList"
                :key="index"
                :value="subject[0]"
              >
                {{ subject[1] }}
              </option>
            </select>
          </div>
          <div class="input-studentScore">
            <label for="subjectSelect">Chọn học kì:</label>
            <select
              class="form-control"
              id="semesterSelect"
              v-model="selectedSemester"
              @change="handleSemesterChange"
            >
              <option value="">Chọn học kì</option>
              <option
                v-for="(semester, index) in semesterList"
                :key="index"
                :value="semester[0]"
              >
                {{ `${semester[1]} - ${semester[2]}` }}
              </option>
            </select>
          </div>
          <div class="input-studentScore">
            <button class="btn btn-primary">Tìm kiếm</button>
          </div>
          <div v-if="isEditMode">
            <div class="input-studentScore">
              <button
                @click="exitHandleEdit"
                class="btn btn-primary"
                style="margin-right: 10px"
              >
                Thoát
              </button>
              <button class="btn btn-primary" style="margin-right: 10px">
                Gửi mail
              </button>
              <button class="btn btn-primary">Đọc file</button>
            </div>
          </div>
          <div v-else>
            <div class="input-studentScore">
              <button @click="handleEdit" class="btn btn-primary">
                Nhập điểm
              </button>
            </div>
          </div>
        </div>
      </form>
      <div v-if="isEditMode" class="form-input-score">
        <div class="table-studentScore">
          <table class="table table-hover">
            <thead>
              <tr class="header-studentScore">
                <th style="width: 150px">Mã số sinh viên</th>
                <th style="width: 20%">Họ và tên</th>
                <th>
                  <div class="form-check">
                    <input
                      type="radio"
                      class="form-check-input"
                      id="radio1"
                      name="optradio"
                      value="option1"
                      v-model="selectedColumn"
                    />Quá trình
                    <label class="form-check-label" for="radio1"></label>
                  </div>
                </th>
                <th>
                  <div class="form-check">
                    <input
                      type="radio"
                      class="form-check-input"
                      id="radio2"
                      name="optradio"
                      value="option2"
                      v-model="selectedColumn"
                    />Giữa kì
                    <label class="form-check-label" for="radio2"></label>
                  </div>
                </th>
                <th>
                  <div class="form-check">
                    <input
                      type="radio"
                      class="form-check-input"
                      id="radio3"
                      name="optradio"
                      value="option3"
                      v-model="selectedColumn"
                    />Cuối kì
                    <label class="form-check-label" for="radio3"></label>
                  </div>
                </th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(student, index) in studentList" :key="index">
                <td style="width: 150px; text-align: center">
                  {{ student.studentId }}
                </td>
                <td style="width: 20%">{{ student.studentName }}</td>
                <td>
                  <input
                    type="number"
                    class="form-control"
                    placeholder=""
                    name="score1"
                    :disabled="selectedColumn !== 'option1'"
                  />
                </td>
                <td>
                  <input
                    type="number"
                    class="form-control"
                    placeholder=""
                    name="score2"
                    :disabled="selectedColumn !== 'option2'"
                  />
                </td>
                <td>
                  <input
                    type="number"
                    class="form-control"
                    placeholder=""
                    name="score3"
                    :disabled="selectedColumn !== 'option3'"
                  />
                </td>
              </tr>
            </tbody>
          </table>
          <div class="btn-save">
            <button class="btn btn-primary">Lưu</button>
          </div>
        </div>
      </div>
      <div v-else>
        <div v-if="studentList.length > 0">
          <table class="table table-striped table-bordered table-hover">
            <thead>
              <tr>
                <th class="text-center">Mã số sinh viên</th>
                <th class="text-center">Tên sinh viên</th>
                <th class="text-center">Ngày sinh</th>
                <th class="text-center">Quá trình</th>
                <th class="text-center">Giữa kì</th>
                <th class="text-center">Cuối kì</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(student, index) in studentList" :key="index">
                <td class="text-center">{{ student.studentId }}</td>
                <td class="text-center">{{ student.studentName }}</td>
                <td class="text-center">
                  {{ formatDate(student.studentBithday) }}
                </td>
                <td class="text-center">
                  {{ getScoreValue(student.scoreDto, "Quá trình") }}
                </td>
                <td class="text-center">
                  {{ getScoreValue(student.scoreDto, "Giữa kì") }}
                </td>
                <td class="text-center">
                  {{ getScoreValue(student.scoreDto, "Cuối kì") }}
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { authApi, endpoints } from "@/configs/Apis.js";
import { mapGetters } from "vuex";
import { useMenu } from "../../../stores/use-menu.js";
export default {
  setup() {
    useMenu().onSelectedKeys(["teacher-studentScore"]);
  },
  data() {
    return {
      isEditMode: false,
      selectedColumn: "option1",
      subjectList: [],
      selectedSubject: "",
      selectedLecturer: {},
      studentList: [],
      semesterList: [],
      selectedSemester: "",
      studentScores: {},
      scoreColumnNames: [],
    };
  },
  computed: {
    ...mapGetters(["isAuth", "getUser"]),
  },
  created() {
    this.fetchLecturerInfo();

    this.fetchLecturerInfo().then((lecturerInfo) => {
      if (lecturerInfo && lecturerInfo.id) {
        const lecturerId = lecturerInfo.id;
        console.log("lecturerId", lecturerId);
        this.fetchSubjectsByLecturerId(lecturerId);
      }
    });
  },
  methods: {
    handleEdit() {
      this.isEditMode = true;
    },
    exitHandleEdit() {
      this.isEditMode = false;
    },
     handleSubjectChange(event) {
      this.selectedSubject = event.target.value;
    },
    handleSemesterChange(event) {
      this.selectedSemester = event.target.value;
    },
    getScoreValue(scoreDto, columnName) {
      const score = scoreDto.find(
        (item) => item.scoreColumnName === columnName
      );
      return score ? score.scoreValue : "";
    },
    formatDate(date) {
      if (!date) return ""; // Tránh xử lý ngày null hoặc undefined

      // Chuyển đối đối tượng ngày sang ngày
      const formattedDate = new Date(date);

      // Lấy thông tin về ngày, tháng và năm
      const day = formattedDate.getDate();
      const month = formattedDate.getMonth() + 1; // Lưu ý: Tháng bắt đầu từ 0
      const year = formattedDate.getFullYear();

      // Định dạng thành chuỗi "ngày/tháng/năm"
      return `${day}/${month}/${year}`;
    },

    
    async fetchLecturerInfo() {
      try {
        const lecturerUsername = this.getUser.username;
        const response = await authApi().get(
          endpoints["get-lecturer-by-username"].replace(
            "{username}",
            lecturerUsername
          )
        );
        console.log(response.data);
        this.selectedLecturer = response.data;

        console.log("this.selectedLecturer", this.selectedLecturer);
        return response.data;
      } catch (error) {
        console.error(err);
        return null;
      }
    },
    async fetchSubjectsByLecturerId(lecturerId) {
      try {
        const response = await authApi().get(
          endpoints["get-subject-by-lecturerId"].replace(
            "{lecturerId}",
            lecturerId
          )
        );
        console.log(response.data);
        this.subjectList = response.data;

        const endpoint = endpoints["semester"] + `?lecturerId=${lecturerId}`;

        const semesterResponse = await authApi().get(endpoint);
        this.semesterList = semesterResponse.data;

        console.log("Semester info:", semesterResponse.data);
      } catch (error) {
        console.error(error);
        console.log(error);
        return null;
      }
    },
    async handleSubmit(event) {
      event.preventDefault();

      try {
        const subjectId = this.selectedSubject;
        const lecturerId = this.selectedLecturer.id;
        const semesterId = this.selectedSemester;
        console.log("semesterId", semesterId);

        const endpoint =
          endpoints["get-list-student"] +
          `?lecturerId=${lecturerId}&subjectId=${subjectId}&semesterId=${semesterId}`;

        const response = await authApi().get(endpoint);

        if (response.data) {
          console.log("get-list-student", response.data);
          this.studentList = response.data;
          console.log("student birthday", this.studentList[0].studentBithday);
        }
      } catch (error) {
        console.error(error);
      }
    },
    
  },
};
</script>