<template>
	<el-col :span="16">
		<el-card shadow="never">
			<el-form :model="value" label-position="top">
				<el-form-item size="small" label="Тип роботи: " prop="workType">
					<el-radio-group v-model="value.semesterType" @change="reset">
						<el-radio label="1">Весняний</el-radio>
						<el-radio label="2">Осінній</el-radio>
					</el-radio-group>
				</el-form-item>
				<el-form-item size="small" prop="extramural" label="Заочний">
					<el-checkbox v-model="value.extramural"></el-checkbox>
				</el-form-item>
				<el-form-item size="small" prop="shortened" label="Скорочений">
					<el-checkbox v-model="value.shortened"></el-checkbox>
				</el-form-item>
				<el-form-item size="small" label="Навчальний рік: ">
					<el-col :span="8">
						<el-form-item prop="beginYear">
							<el-input-number size="mini" v-model="value.beginYear" style="width: 100%;"
							                 @change="onBeginYearChanged" :min="1930" :max="2070"/>
						</el-form-item>
					</el-col>
					<el-col class="line" :span="2">-</el-col>
					<el-col :span="8">
						<el-form-item prop="endYear">
							<el-input-number size="mini" v-model="value.endYear" style="width: 100%;"
							                 @change="onEndYearChanged" :min="1931" :max="2071"/>
						</el-form-item>
					</el-col>
				</el-form-item>
				<el-form-item size="small" prop="courseNumber" label="Курс: ">
					<el-input-number placeholder="Pick a time" size="small"
				                 v-model="value.courseNumber"
				                 :min="1" :max="6"
				                 @change="onCourseNumberChanged"/>
				</el-form-item>
				<el-form-item size="small" prop="group" label="Група: ">
					<el-select size="small" v-model="value.group"
					           remote filterable
					           :loading="isLoading"
					           loading-text="Зачекайте доки їде запит."
					           :remote-method="findGroupsByParam"
					           @change="onGroupChange"
					           value-key="id" style="width: 100%">
						<el-option v-for="group in groupSuggestions" :key="group.id"
						           :value="group" :label="group.groupName"/>
					</el-select>
				</el-form-item>
				<el-form-item size="small" prop="fullName" label="ПІБ: ">
					<el-input size="small" v-model="value.fullName"/>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" @click.native.stop.prevent="send">{{ value.id === 0 ? "Додати" : "Відновити" }}</el-button>
				</el-form-item>
			</el-form>
		</el-card>
	</el-col>
</template>

<script lang="tsx">
	import Vue from "vue"
	import { Component, Prop, Watch } from "vue-property-decorator"
	import {DisciplineView, GroupView, QualificationWorkFormData, StudentView, TeacherView, 
		validateArrayRequired, validateObjectRequired, validateRequired, validateString} from "../entities/entities"
	import { Student } from "@/entities/entities"

	@Component({
		name: "StudentsForm"
	})
	export default class StudentsForm extends Vue {
		
		private isLoading: boolean = false

		private value: Student = {
			id: 0,
			fullName: "",
			beginYear: 2018,
			endYear: 2019,
			semesterType: "1",
			group: {
				id: 0,
				groupName: ""
			},
			courseNumber: 1,
			extramural: false,
			shortened: false
		}

		private groupSuggestions: GroupView[] = []

		private onBeginYearChanged(val: number) {
			this.value.endYear = val + 1
			this.reset()
		}

		private onEndYearChanged(val: number) {
			this.value.beginYear = val - 1
			this.reset()
		}

		private onCourseNumberChanged(val: number, oldVal: number) {
			this.reset()
		}

		private onGroupChange(groupView: GroupView) {
			this.groupSuggestions = [groupView]
		}

		private reset() {
			this.value.group = {
				id: 0,
				groupName: ""
			}
		}

		private findGroupsByParam(groupNamePart: string) {
			if (groupNamePart === "") {
				return
			}
			this.isLoading = true
			this.axios.post("/group/get-by-course-number-and-academic-year", {
				beginYear: this.value.beginYear,
				endYear: this.value.endYear,
				courseNumber: this.value.courseNumber,
				semester: this.value.semesterType,
				extramural: this.value.extramural,
				shortened: this.value.shortened,
				groupNamePart
			}).then((value) => {
				this.isLoading = false
				this.groupSuggestions = value.data.groups
			}).catch((reason) => {
				console.log(reason)
			})
		}

		private send() {
			const errors = this.validate()

			if (errors.length > 0) {
				this.$msgbox({
					message: <span style="white-space: pre-line;">
						{ errors }
					</span>,
					type: "error",
					confirmButtonText: "OK",
					title: "Помилка"
				})

				return
			}

			if (this.value.id === 0) {
				this.insert()
			} else {
				this.update()
			}
		}

		private update() {
			
		}

		private insert() {
			this.axios.put("/student/insert", this.value).then((resp) => {
				if (resp.data.successful) {
					this.$alert("Студент успішно додан", "Успіх", {type: "success"})
				} else {
					this.$alert("З'явилася якась помилка", "Помилка", {type: "error"})
				}
			})
		}

		private validate(): string {
			let errors = ""
			
			if (validateObjectRequired(this.value.group, {id: 0, groupName: ""}) !== "fine") {
				errors += "Ви не вибрали групу\n"
			}

			if (validateString(this.value.fullName) !== "fine") {
				errors += "Ви не ввели ПІБ студента\n"
			}

			return errors
		}
	}
</script>

<style scoped lang="scss">
	.line {
		text-align: center;
	}
</style>
