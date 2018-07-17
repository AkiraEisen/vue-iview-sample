<template>
	<!-- 这里不能用this.modalShow,不知为什么，报错！ -->
	<Modal v-model="modalShow" width="1000" title="组管理">
		<Table :columns="this.columns" :data="this.data">
		</Table>
		<div slot="footer">
			<Button @click="this.cancelHandler" size="small" icon="close-round">关闭</Button>
			<!--<Button type="success" @click="this.okHandler" size="small" icon="checkmark-round">确定</Button>-->
			<Button type="primary" @click="this.createHandler" size="small" icon="plus-round">新增</Button>
		</div>
	</Modal>
</template>

<script>
  export default {
    name: 'group',
    data () {
      return {
        editMode: false,
        currentRowIndex: -1,
        backupRow: {},
        columns: [
          {
            title: 'ID',
            key: 'id',
            align: 'center'
          },
          {
            title: '名称',
            key: 'name',
            align: 'center',
            render: (h, params) => {
              return this.checkEdit(params)
                ? h('div', [
                  h('Input', {
                    props: {
                      value: this.data[params.index].name
                    },
                    on: {
                      input: (val) => {
                        params.row.name = val;
                      }
                    }
                  })
                ])
                : h('div', [
                  this.data[params.index].name
                ]);
            }
          },
          {
            title: '展示名称',
            key: 'name_cn',
            align: 'center',
            render: (h, params) => {
              return this.checkEdit(params)
                ? h('div', [
                  h('Input', {
                    props: {
                      value: this.data[params.index].name_cn
                    },
                    on: {
                      input: (val) => {
                        params.row.name_cn = val;
                      }
                    }
                  })
                ])
                : h('div', [
                  this.data[params.index].name_cn
                ]);
            }
          },
          {
            title: '国际化',
            key: 'international_key',
            align: 'center',
            render: (h, params) => {
              return this.checkEdit(params)
                ? h('div', [
                  h('Input', {
                    props: {
                      value: this.data[params.index].international_key
                    },
                    on: {
                      input: (val) => {
                        params.row.international_key = val;
                      }
                    }
                  })
                ])
                : h('div', [
                  this.data[params.index].international_key
                ]);
            }
          },
          {
            title: '排序',
            key: 'seq',
            align: 'center',
            render: (h, params) => {
              return this.checkEdit(params)
                ? h('div', [
                  h('Input', {
                    props: {
                      value: this.data[params.index].seq
                    },
                    on: {
                      input: (val) => {
                        params.row.seq = val;
                      }
                    }
                  })
                ])
                : h('div', [
                  this.data[params.index].seq
                ]);
            }
          },
          {
            title: '操作',
            key: 'action',
            width: 160,
            align: 'center',
            fixed: 'right',
            render: (h, params) => {
              return this.checkEdit(params)
                ? h('div', [
                  h('Button', {
                    props: {
                      type: 'success',
                      size: 'small'
                    },
                    on: {
                      click: () => {
                        this.data[this.currentRowIndex] = Object.assign({}, params.row);
                        this.submitData(this.update).then(() => {
                          this.currentRowIndex = -1;
                          this.editMode = false;
                        });
                      }
                    }
                  }, '确定')
                ])
                : h('div', [
                  h('Button', {
                    props: {
                      type: 'text',
                      size: 'small'
                    },
                    on: {
                      click: () => {
                        this.update = true;
                        this.recoverRow();
                        this.currentRowIndex = params.index;
                        this.backupRow = Object.assign({}, this.data[params.index]);
                        this.editMode = true;
                      }
                    }
                  }, '编辑'),
                  h('Button', {
                    props: {
                      type: 'text',
                      size: 'small'
                    },
                    on: {
                      click: () => {
                        this.$http.delete(`/core/extension/defination/formgroups/${params.row.id}`).then(() => {
                          this.queryGroup().then(() => {
                            this.$Message.success('删除成功！');
                          });
                        })
                      }
                    }
                  }, '删除')
                ]);
            }
          }
        ],
        data: [],
        update: false
      };
    },
    methods: {
      // okHandler () {
      //   // 隐藏
      //   this.recoverRow();
      //   this.initializeEdit();
      //   this.$store.commit('dm_entity_action_group', false);
      // },
      cancelHandler () {
        // 隐藏
        this.recoverRow();
        this.initializeEdit();
        this.$store.commit('dm_entity_action_group', false);
      },
      createHandler () {
        this.update = false;
        this.submitData(this.update);
      },
      submitData (update) {
        let entity = this.$store.state.dm.entity_current_object;
        console.log(this.data[this.currentRowIndex]);
        let promise = new Promise((resolve, reject) => {
          this.$http({
            url: '/core/extension/defination/formgroups',
            method: update ? 'put' : 'post',
            data: update ? this.data[this.currentRowIndex] : {entity_id: entity.id}
          }).then(res => {
            this.queryGroup().then(() => {
              this.$Message.success(update ? '实体更新成功！' : '实体创建成功！');
              resolve();
            });
          });
        });
        return promise;
      },
      checkEdit (params) {
        return this.editMode && params.index === this.currentRowIndex;
      },
      recoverRow () {
        this.currentRowIndex === -1 ? '' : this.data[this.currentRowIndex] = Object.assign({}, this.backupRow);
      },
      initializeEdit () {
        this.currentRowIndex = -1;
        this.editMode = false;
        this.backupRow = {};
      },
      queryGroup () {
        let entity = this.$store.state.dm.entity_current_object;
        return this.$http.get(`/core/extension/defination/entities/${entity.id}/formgroups`).then(res => {
          this.data = res.data.result;
          return new Promise((resolve, reject) => {
            resolve();
          });
        });
      }
    },
    computed: {
      modalShow: {
        get () {
          return this.$store.state.dm.entity_action_group;
        },
        set () {
          // 通过on-ok、on-cancel 事件主动设置modal状态。
          this.$store.commit('dm_entity_action_group', false);
        }
      }
    },
    created () {
      this.queryGroup();
    }
  };
</script>

<style scoped>

</style>
