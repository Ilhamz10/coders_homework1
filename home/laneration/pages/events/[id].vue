<template>
	<Header
		:pageName="'Мероприятие #' + route.params.id"
		pageAbbr="events"
		:user="user" />
	<div class="main-content">
		<div class="page-content">
			<div class="header-bg">
				<div class="container-fluid">
					<div class="row">
						<div class="col-12 pt-5 mt-4">
							<div
								id="morris-bar-example"
								class="morris-charts morris-chart-height"></div>

							<!--
                          <div class="mt-4 text-center">
                              <button type="button"
                                  class="btn btn-outline-primary ms-1 waves-effect waves-light">Year 2017</button>
                              <button type="button"
                                  class="btn btn-outline-info ms-1 waves-effect waves-light">Year 2018</button>
                              <button type="button"
                                  class="btn btn-outline-primary ms-1 waves-effect waves-light">Year 2019</button>
                          </div>
                          -->
						</div>
					</div>
				</div>
			</div>

			<div class="container-fluid">
				<div class="row">
					<div class="col-lg-6 grid-margin stretch-card">
						<div class="card">
							<div class="card-body">
								<h4 class="card-title">Неизменяемая информация</h4>
								<table class="table">
									<thead>
										<tr>
											<th></th>
											<th></th>
										</tr>
									</thead>
									<tbody>
										<tr>
											<td>Название</td>
											<td style="text-align: right">
												{{ getEventList[0].title }}
											</td>
										</tr>
										<tr>
											<td>Город</td>
											<td style="text-align: right">
												{{ getEventList[0].cityName }}
											</td>
										</tr>
										<tr>
											<td>Место проведения</td>
											<td style="text-align: right">
												{{ getEventList[0].placeName }}
											</td>
										</tr>
										<tr>
											<td>Метод сбора</td>
											<td style="text-align: right">
												На баланс <b>Личного кабинета</b>
											</td>
										</tr>
										<tr>
											<td>Возрастное ограничение</td>
											<td style="text-align: right">+16</td>
										</tr>
									</tbody>
								</table>
								<div class="row p-2">
									<button id="removeEvent" class="mt-2 btn btn-danger">
										Удалить мероприятие
									</button>
								</div>
							</div>
						</div>
					</div>
					<div class="col-lg-6 grid-margin stretch-card">
						<div class="card">
							<div class="card-body">
								<table class="table">
									<thead>
										<tr>
											<th></th>
											<th></th>
										</tr>
									</thead>
									<tbody>
										<tr>
											<td>Состояние мероприятия</td>
											<td style="text-align: right">Активно</td>
										</tr>
										<tr>
											<td>Тип лиги</td>
											<td style="text-align: right">КОНТИНЕНТАЛЬНАЯ ХЛ</td>
										</tr>
										<tr>
											<td>Выступают</td>
											<td style="text-align: right">
												<div class="teams">
													<img
														:src="getTeamsList[0].logo"
														height="32"
														width="32" />
													Химик против ЦСКА
													<img
														:src="getTeamsList[1].logo"
														height="32"
														width="32" />
												</div>
											</td>
										</tr>
										<tr>
											<td>Теги</td>
											<td style="text-align: right">(Array) Тест, Тест2</td>
										</tr>
									</tbody>
								</table>
								<div class="row p-2">
									<button id="saveEvent" class="btn btn-success">
										Сохранить
									</button>
								</div>
							</div>
						</div>
					</div>
					<div class="col-lg-12 grid-margin stretch-card">
						<div class="card" v-if="loadedData != 0">
							<div class="card-body">
								<!--<h4 class="card-title">Настройка схемы</h4>-->
								<div class="row">
									<div class="col-lg-2">
										<div class="row">
											<div class="col-lg-12">
												<div class="btn-group">
													<button class="btn btn-primary" id="addGroup">
														Добавить группу
													</button>
													<button class="ms-2 btn btn-success" id="saveGroups">
														Сохранить группы
													</button>
												</div>
											</div>
										</div>
										<ul class="selGroup"></ul>
									</div>
									<div class="col-lg-10">
										<Scheme :mode="mType" :id="schemeId" :groups="groups" />
									</div>
								</div>
							</div>
						</div>
					</div>
				</div>
			</div>
			<Footer />
			<!-- Modal -->
			<div class="modal fade" id="oldVersion" tabindex="-1" aria-hidden="true">
				<div class="modal-dialog modal-dialog-centered">
					<div class="modal-content">
						<div class="modal-header">
							<h5 class="modal-title" id="exampleModalLabel">
								Устаревшая версия
							</h5>
						</div>
						<div class="modal-body">
							<form @submit.prevent="onDelete()">
								<div class="mb-3">
									<div class="alert alert-danger">
										Версия схемы в данном мероприятии - устаревшая!<br />
										Необходимо удалить мероприятие и создать новое с актуальными
										данными схемы...
									</div>
								</div>
								<button
									type="submit"
									id="deleteEvent"
									class="btn btn-primary"
									style="width: 100%">
									Удалить мероприятие
								</button>
							</form>
						</div>
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<style>
.selGroup {
	list-style: none;
	display: flex;
	flex-direction: column;
	padding: 0;
	margin: 0;
	margin-top: 30px;
}

.selGroup li {
	border: 1px solid #d5d5d5;
	padding: 10px;
	margin: 10px;
}

.selGroup li div {
	margin: 5px 0px 5px 0px;
}

.teams img {
	margin: 0px 5px 0px 5px;
}
</style>

<script setup>
const mType = 'event';
var schemeId = -1;

const route = useRoute();

let loadedData = 0;
let getEventList = [];
let getTeamsList = [];
let getGroupsList = [];

var groups = {
	selected: 0,
	edit: false,
	active: -1,
	list: [],
};

var editGroupId = -1;

useHead({
	titleTemplate: (title) => `Мероприятие №${route.params.id}`,
	viewport: 'width=device-width, initial-scale=1, maximum-scale=1',
	charset: 'utf-8',
	bodyAttrs: {
		class: 'app',
	},
	link: [],
	script: [],
});

await useFetch(`https://ticketscms.tellarion.dev/api/events/detail`, {
	params: { jwt: useCookie('jwt').value, eventId: route.params.id },
	method: 'POST',
}).then(async (dd) => {
	let logicData = JSON.parse(dd.data.value);

	getEventList = logicData.data.detail;
	getGroupsList = logicData.data.groups;
	getTeamsList = logicData.data.teams;

	loadedData = logicData.data.length;

	schemeId = logicData.data.place[0].schemeId;

	if (loadedData <= 0) {
		return navigateTo('/events', { redirectCode: 301 });
	}

	// go

	async function removeEvent(eventId) {
		await useFetch(`https://ticketscms.tellarion.dev/api/events/remove`, {
			params: { jwt: useCookie('jwt').value, eventId: eventId },
			method: 'POST',
		}).then(async (dd) => {
			window.location.reload();
		});
	}

	// тут крч отдых
	async function saveGroups(groupsList, eventId) {
		console.log(groupsList);
		await useFetch(`https://ticketscms.tellarion.dev/api/events/groups/save`, {
			params: {
				jwt: useCookie('jwt').value,
				eventId: eventId,
				groups: encode(JSON.stringify(groupsList)),
			},
			method: 'POST',
		}).then(async (dd) => {
			window.location.reload();
		});
	}

	$(document).ready(function () {
		if (
			logicData.data.detail[0].schemeBuild != logicData.data.detail[0].build
		) {
			const myModal = new bootstrap.Modal('#oldVersion', {
				backdrop: 'static',
				keyboard: false,
			});
			myModal.show();
			$('#deleteEvent').on('click', async function () {
				removeEvent(route.params.id);
			});
		}

		$('#removeEvent').on('click', function () {
			removeEvent(route.params.id);
		});

		$('#saveEvent').on('click', function () {
			window.location.reload();
		});

		// groups write

		for (let i = 0; i < getGroupsList.length; i++) {
			let getPlacesGroups = JSON.parse(getGroupsList[i].places);
			groups.active = {
				id: getGroupsList[0].id,
				color: getGroupsList[0].color,
			};
			groups.edit = false;
			groups.list.push({
				id: getGroupsList[i].id,
				comment: getGroupsList[i].comment,
				color: getGroupsList[i].color,
				coast: getGroupsList[i].coast,
				places: getPlacesGroups,
				selected: getPlacesGroups.length,
				edit: false,
			});
		}
		manageFormGroup('load', $('.selGroup'), groups);

		function manageFormGroup(type, parent, items, id) {
			if (type === 'load') {
				parent.html('');
				for (let group of items.list) {
					let liElem = '<li>';
					liElem += `<form data-group="${group.id}">
		            <div class="name"><label>Название: ${group.comment}</label></div>
		            <div class="count"><label>Выбранное количество: <span>${group.selected}</span></label></div>
		            <div class="price"><label>Цена: ${group.coast}</label></div>
		            <div class="color"><label>Цвет: ${group.color} <div class="previewColorMin" style="background-color: ${group.color}; height: 16px; width: 16px; display: inline-block;"></div></div>
		            <div class="actions"><div class="row"><button class="btn btn-primary editGroup">Редактировать</div></div>
		          </form>`;
					liElem += '</li>';
					parent.append(liElem);
				}
			}

			if (type === 'edit') {
				const currentGroup = items.list.find((group) => group.id === id);
				let liElem = `<form data-group="${id}">
		            <div class="name"><label>Название</label><input type="text" id="gTitle" class="form-control" placeholder="${currentGroup.comment}" value="" /></div>
		            <div class="count"><label>Выбранное количество: <span>${currentGroup.selected}</span></label></div>
		            <div class="price"><label>Цена</label><input type="number" class="form-control" id="gCoast" value="${currentGroup.coast}"/></div>
		            <div class="color"><label>Цвет (hex, name)</label><input type="text" class="form-control" id="gColor" onChange="updateColor(this.jscolor);" data-jscolor="${currentGroup.color}" value="${currentGroup.color}"/></div>
		            <div class="actions"><div class="row"><button class="btn btn-success" id="saveGroup">Сохранить</div> <div class="row"><button class="btn btn-danger" id="removeGroup">Удалить</div></div></div>
		          </form>`;

				parent.innerHTML = liElem;
			}
		}

		$('.selGroup').click(function (e) {
			if (e.target.innerText.includes('Редактировать')) {
				e.preventDefault();
				let parent = e.target.parentNode.parentNode.parentNode.parentNode;
				let id = Number(
					e.target.parentNode.parentNode.parentNode.getAttribute('data-group')
				);
				let isThereEditedGroup = groups.list.some((group) => group.edit);
				const currentGroup = groups.list.find((group) => group.id === id);
				groups.active = {
					id: currentGroup.id,
					color: currentGroup.color,
				};
				if (!isThereEditedGroup) {
					currentGroup.edit = true;
					manageFormGroup('edit', parent, groups, id);
				} else {
					alert('Please firstly save changes');
				}

				console.log(groups);
			}

			if (e.target.innerText.includes('Сохранить')) {
				e.preventDefault();
				let id = Number(
					e.target.parentNode.parentNode.parentNode.getAttribute('data-group')
				);
				const currentGroup = groups.list.find((group) => group.id === id);
				currentGroup.comment =
					$(`form[data-group=${id}] #gTitle`).val().trim() === ''
						? currentGroup.comment
						: $(`form[data-group=${id}] #gTitle`).val().trim();
				currentGroup.coast = $(`form[data-group=${id}] #gCoast`).val();
				currentGroup.color = $(`form[data-group=${id}] #gColor`).val();
				currentGroup.edit = false;
				manageFormGroup('load', $('.selGroup'), groups);
			}

			if (e.target.innerText.includes('Удалить')) {
				e.preventDefault();
				let id = Number(
					e.target.parentNode.parentNode.parentNode.getAttribute('data-group')
				);
				groups = {
					...groups,
					list: groups.list.filter((group) => group.id !== id),
				};
				manageFormGroup('load', $('.selGroup'), groups);
			}
		});

		$('#addGroup').click(function () {
			let randomColor = `#${Math.floor(Math.random() * 16777215).toString(16)}`;
			let id = groups.list.length > 0 ? groups.list.at(-1).id + 1 : 1;

			groups.list.unshift({
				id,
				comment: `Новая группа #${id + 1}`,
				color: randomColor,
				coast: 1000,
				places: [],
				selected: 0,
				edit: false,
			});
			manageFormGroup('load', $('.selGroup'), groups);
		});

		$('#saveGroups').on('click', function () {
			if (groups.list.length >= 0) {
				saveGroups(groups.list, route.params.id);
			} else {
				alert('Недостаточно групп для сохранения');
			}
		});

		console.log(groups);

		//

		// function manageFormGroup(type, element, groups, editValue) {
		// 	let activeIdx =
		// 		type == 'new' || type == 'load' ? groups.active : editValue;
		// 	let tplGroupEdit = ``;
		// 	if (type == 'new' || type == 'load') tplGroupEdit += `<li>`;

		// 	if (type == 'new' || type == 'edit')
		// 		tplGroupEdit += `
		//           <form data-group="${activeIdx}">
		//             <div class="name"><label>Название</label><input type="text" id="gTitle" class="form-control" placeholder="${groups.list[activeIdx].comment}" value="" /></div>
		//             <div class="count"><label>Выбранное количество: <span>${groups.list[activeIdx].selected}</span></label></div>
		//             <div class="price"><label>Цена</label><input type="number" class="form-control" id="gCoast" value="${groups.list[activeIdx].coast}"/></div>
		//             <div class="color"><label>Цвет (hex, name)</label><input type="text" class="form-control" id="gColor" onChange="updateColor(this.jscolor);" data-jscolor="${groups.list[activeIdx].color}" value="${groups.list[activeIdx].color}"/></div>
		//             <div class="actions"><div class="row"><button class="btn btn-success" id="saveGroup">Сохранить</div> <div class="row"><button class="btn btn-danger" id="removeGroup">Удалить</div></div></div>
		//           </form>
		//         `;

		// 	if (type == 'load')
		// 		tplGroupEdit += `
		//           <form data-group="${activeIdx}">
		//               <div class="name"><label>Название: ${groups.list[activeIdx].comment}</label></div>
		//               <div class="count"><label>Выбранное количество: <span>${groups.list[activeIdx].selected}</span></label></div>
		//               <div class="price"><label>Цена: ${groups.list[activeIdx].coast}</label></div>
		//               <div class="color"><label>Цвет: ${groups.list[activeIdx].color} <div class="previewColorMin" style="background-color: ${groups.list[activeIdx].color}; height: 16px; width: 16px; display: inline-block;"></div></div>
		//               <div class="actions"><div class="row"><button class="btn btn-primary editGroup">Редактировать</div></div>
		//           </form>
		//         `;

		// 	if (type == 'new' || type == 'load') tplGroupEdit += `</li>`;

		// 	if (type == 'new' || type == 'load') element.append(tplGroupEdit);

		// 	if (type == 'edit') element.html(tplGroupEdit);

		// 	if (type == 'remove') element.remove();

		// 	if (type == 'new' || type == 'edit') {
		// 		$('#gTitle').off();
		// 		$('#gTitle').on('change', function () {
		// 			if ((groups.edit = true)) {
		// 				groups.list[activeIdx].comment = $(this).val();
		// 			}
		// 		});

		// 		$('#gCoast').off();
		// 		$('#gCoast').on('change', function () {
		// 			if ((groups.edit = true)) {
		// 				groups.list[activeIdx].coast = $(this).val();
		// 			}
		// 		});

		// 		$('#gColor').off();
		// 		$('#gColor').on('change', function () {
		// 			let vColor = $(this).val();
		// 			if ((groups.edit = true)) {
		// 				groups.list[activeIdx].color = vColor;
		// 			}
		// 		});

		// 		jscolor.install();

		// 		jscolor.trigger('input change');

		// 		$('#saveGroup').off();
		// 		$('#saveGroup').on('click', function (e) {
		// 			e.preventDefault();
		// 			console.log('save data');
		// 			console.log(groups);
		// 			$(`.selGroup [data-group='${activeIdx}']`).html(`
		//               <form data-group="${activeIdx}">
		//                 <div class="name"><label>Название: ${groups.list[activeIdx].comment}</label></div>
		//                 <div class="count"><label>Выбранное количество: <span>${groups.list[activeIdx].selected}</span></label></div>
		//                 <div class="price"><label>Цена: ${groups.list[activeIdx].coast}</label></div>
		//                 <div class="color"><label>Цвет: ${groups.list[activeIdx].color} <div class="previewColorMin" style="background-color: ${groups.list[activeIdx].color}; height: 16px; width: 16px; display: inline-block;"></div></div>
		//                 <div class="actions"><div class="row"><button class="btn btn-primary editGroup">Редактировать</div></div>
		//               </form>
		//             `);
		// 			groups.edit = false;
		// 			groups.active = -1;
		// 			//$('#return').click()

		// 			$('.editGroup').off();
		// 			$('.editGroup').on('click', function (e) {
		// 				e.preventDefault();
		// 				if (groups.edit == false) {
		// 					let getValue = $(this).parent().parent().parent().data('group');
		// 					groups.active = getValue;
		// 					groups.edit = true;
		// 					manageFormGroup(
		// 						'edit',
		// 						$(this).parent().parent().parent(),
		// 						groups,
		// 						getValue
		// 					);
		// 				}
		// 			});
		// 		});

		// 		$('#removeGroup').off();
		// 		$('#removeGroup').on('click', function (e) {
		// 			e.preventDefault();
		// 			let getValue = $(this).parent().parent().parent().data('group');
		// 			groups.list.splice(groups.active, 1);
		// 			groups.active = -1;
		// 			groups.edit = false;
		// 			manageFormGroup(
		// 				'remove',
		// 				$(this).parent().parent().parent().parent(),
		// 				groups,
		// 				getValue
		// 			);
		// 			$('#return').click();
		// 			// fix after
		// 			//saveGroups(groups.list, route.params.id)
		// 			//window.location.reload()
		// 		});
		// 	}

		// 	if (type == 'load') {
		// 		$('.editGroup').off();
		// 		$('.editGroup').on('click', function (e) {
		// 			e.preventDefault();
		// 			if (groups.edit == false) {
		// 				let getValue = $(this).parent().parent().parent().data('group');
		// 				groups.active = getValue;
		// 				groups.edit = true;
		// 				manageFormGroup(
		// 					'edit',
		// 					$(this).parent().parent().parent(),
		// 					groups,
		// 					getValue
		// 				);
		// 			}
		// 		});
		// 	}
		// }

		// $('#addGroup').on('click', function () {
		// 	if (groups.edit == false) {
		// 		groups.active = groups.list.length;
		// 		groups.edit = true;

		// 		let randomColor = `#${Math.floor(Math.random() * 16777215).toString(
		// 			16
		// 		)}`;

		// 		groups.list.push({
		// 			comment: `Новая группа #${groups.active + 1}`,
		// 			color: randomColor,
		// 			coast: 1000,
		// 			places: [],
		// 			selected: 0,
		// 		});

		// 		manageFormGroup('new', $('.selGroup'), groups);

		// 		console.log('ID GROUP');
		// 		console.log(groups.active);
		// 	} else {
		// 		alert('Закончите редактирование группы для создания новой');
		// 	}
		// });

		// $('#saveGroups').on('click', function () {
		// 	if (groups.list.length >= 0) {
		// 		saveGroups(groups.list, route.params.id);
		// 	} else {
		// 		alert('Недостаточно групп для сохранения');
		// 	}
		// });
	});
});
</script>

<script>
import { encode, decode } from 'hi-base64';

import Header from '@/components/Header.vue';
import Footer from '@/components/Footer.vue';

import Scheme from '@/components/Scheme.vue';

export default {
	components: {
		Header: Header,
		Footer: Footer,
		Scheme: Scheme,
	},
	props: ['user'],
	data() {
		return {};
	},
	methods: {
		updateColor(picker) {
			let getAllElements = document.getElementsByClassName('place');
			getAllElements.forEach((element) => {
				element.style.background = picker.toBackground();
			});
		},
	},
};
</script>
