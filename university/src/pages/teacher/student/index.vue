<template>
 
  <div class="studentHome">
    <form @submit.prevent="handleSubmit">
      <div class="select">
        <div class="form-group" :class="{ 'has-error': !selectedSubject }">
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
        <div class="semester">
          <div class="form-group" :class="{ 'has-error': !selectedSemester }">
            <label for="semesterSelect">Chọn học kì:</label>
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
        </div>
        <button class="btn btn-primary btnSubmit" type="submit">
          Tìm kiếm
        </button>
      </div>
      <!-- <div class="btnFile">
        <button
          class="btn btn-success btnExport"
          type="button"
          @click="handleExportCSV"
        >
          Xuất file
        </button>
        <button class="btn btn-info btnImport" type="button">
          <label for="csvFile" class="whiteColor" style="margin: 0"
            >Đọc file</label
          >
          <input
            id="csvFile"
            type="file"
            accept=".csv"
            style="display: none"
            @change="handleFileChange"
          />
        </button>
        <button class="btn btn-danger btnExportPDF" @click="exportToPDF">
          Xuất PDF
        </button>
      </div> -->
    </form>
    <!-- <div v-if="csvData.length > 0">
      <h3>Dữ liệu từ tệp CSV</h3>
      <table class="table table-striped table-bordered table-hover">
        <thead>
          <tr>
            <th
              class="text-center"
              v-for="(column, index) in Object.keys(csvData[0])"
              :key="index"
            >
              {{ column }}
            </th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(row, rowIndex) in csvData" :key="rowIndex">
            <td
              class="text-center"
              v-for="(value, valueIndex) in Object.values(row)"
              :key="valueIndex"
            >
              {{ value }}
            </td>
          </tr>
        </tbody>
      </table>
    </div> -->
    <div v-if="studentList.length > 0">
      <table class="table table-striped table-bordered table-hover">
        <thead>
          <tr>
            <th class="text-center">Mã số sinh viên</th>
            <th class="text-center">Tên sinh viên</th>
            <th
              class="text-center"
              v-for="(scoreColumn, index) in scoreColumns"
              :key="index"
            >
              {{ scoreColumn }}
            </th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(student, index) in studentScores" :key="index">
            <td class="text-center">{{ student.studentId }}</td>
            <td class="text-center">{{ student.studentName }}</td>
            <td
              class="text-center"
              v-for="(score, scoreIndex) in student.scores"
              :key="scoreIndex"
            >
              {{ score.value }}
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
import { authApi, endpoints } from "@/configs/Apis.js";
import { mapGetters } from "vuex";
import { useMenu } from "../../../stores/use-menu.js";

export default {
  setup() {
    useMenu().onSelectedKeys(["teacher-student"]);
  },
  computed: {
    ...mapGetters(["isAuth", "getUser"]),
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
    //   csvData: [],
      studentScores: {},
      scoreColumns: [],
    };
  },
  created() {
    this.fetchLecturerInfo();
    // this.fetchData();
    
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
    handleSubjectChange(event) {
      this.selectedSubject = event.target.value;
    },
    handleSemesterChange(event) {
      this.selectedSemester = event.target.value;
    },
    async handleSubmit(event) {
      event.preventDefault();

      try {
        const subjectId = this.selectedSubject;
        const lecturerId = this.selectedLecturer.id;
        const semesterId = this.selectedSemester;
        console.log("semesterId", semesterId);

        const endpoint =
          endpoints["get-list-scores"] +
          `?lecturerId=${lecturerId}&semesterId=${semesterId}&subjectId=${subjectId}`;

        const response = await authApi().get(endpoint);

        if (response.data) {
          console.log(response.data);
          this.studentList = response.data;
        }
      } catch (error) {
        console.error(error);
      }
    },
  },
  watch: {
      studentList: {
        handler: function (newVal) {
          // Process the student list and update studentScores and scoreColumns
          const studentScores = {};
          const scoreColumns = [];

          newVal.forEach((student) => {
            const { studentId, studentName, scoreColumnName, scoreValue } =
              student;

            if (!studentScores[studentId]) {
              studentScores[studentId] = {
                studentId,
                studentName,
                scores: [{ column: scoreColumnName, value: scoreValue }],
              };
            } else {
              studentScores[studentId].scores.push({
                column: scoreColumnName,
                value: scoreValue,
              });
            }

            if (!scoreColumns.includes(scoreColumnName)) {
              scoreColumns.push(scoreColumnName);
            }
          });

          this.studentScores = studentScores;
          console.log(studentScores);
          this.scoreColumns = scoreColumns;
          console.log(studentScores);
        },
        deep: true,
      },
    },
};
</script>