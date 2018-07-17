<template>
	<Card class="entity-detail-card" ref="entityDetailCard">
		<span slot="title"><h2>实体定义{{this.currentEntity.name ? `(${this.currentEntity.name})` : ''}}</h2></span>
		<span slot="extra">
			<Button type="ghost" size='small' style="margin-left: 5px;" @click="this.backward">返回</Button>
			<Button v-if="this.editMode"
			        type="primary" size='small' style="margin-left: 5px;"
			        @click="this.save">保存
			</Button>
			<Button v-if="this.editMode"
			        type="primary" size='small' style="margin-left: 5px;"
			        @click="this.groupManage">组管理
			</Button>
		</span>
		<!-- entity defination form -->
		<Form :label-width="75" :model="currentEntity">
			<Row>
				<Col span="12">
					<FormItem label="ID" prop="id">
						<Input size="small" type="text" clearable v-model="currentEntity.id" :disabled="true"></Input>
					</FormItem>
				</Col>
				<Col span="12">
					<FormItem label="实体类型" prop="type">
						<Select size="small" v-model="currentEntity.type" :disabled="!editMode || !this.filedEditable">
							<Option value="00">
								实体
							</Option>
							<Option value="10">
								扩展实体
							</Option>
							<Option value="11">
								扩展实体(明细)
							</Option>
						</Select>
					</FormItem>
				</Col>
			</Row>
			<Row>
				<Col span="12">
					<FormItem label="名称" prop="name">
						<Input size="small" type="text" clearable v-model="currentEntity.name" :disabled="!editMode || !this.filedEditable"></Input>
					</FormItem>
				</Col>
				<Col span="12">
					<FormItem label="展示名称" prop="name_cn">
						<Input size="small" type="text" clearable v-model="currentEntity.name_cn" :disabled="!editMode"></Input>
					</FormItem>
				</Col>
			</Row>
			<Row>
				<Col span="4">
					<FormItem label="状态" prop="status">
						<Select size="small" v-model="currentEntity.status" :disabled="true">
							<Option value="01">
								编辑
							</Option>
							<Option value="03">
								发布
							</Option>
							<Option value="04">
								废弃
							</Option>
						</Select>
					</FormItem>
				</Col>
				<Col span="8">
					<FormItem label="连接ID" prop="conn_id">
						<Input size="small" type="text" clearable v-model="currentEntity.conn_id" :disabled="!editMode"></Input>
					</FormItem>
				</Col>
				<Col span="12">
					<FormItem label="实体描述" prop="description">
						<Input size="small" type="text" clearable v-model="currentEntity.description"
						       :disabled="!editMode"></Input>
					</FormItem>
				</Col>
			</Row>
		</Form>
		<!-- entity fileds defination table-->

		<Row type="flex" justify="end" class="code-row-bg">
			<Col span="1">
				<div style="float: right">
					<Button v-if="this.editMode && this.currentEntity.id"
					        type="primary"
					        size='small'
					        style="margin-bottom: 5px"
					        @click="this.create">新增
					</Button>
					<!--<span style="margin-bottom: 5px">-->
					<h3 v-if="!this.currentEntity.id" style="width: 270px; margin-bottom: 5px">请先保存，再次编辑时可新增字段！</h3>
					<!--</span>-->
				</div>
			</Col>
		</Row>

		<Table
			:loading="this.tableLoading"
			:border="true"
			size="small"
			:stripe="true"
			:columns="columns"
			:data="fields"
			:height="tableHeight"></Table>

		<field-detail @refresh-fields="this.queryFields"></field-detail>

		<group></group>
		<Modal v-model="deleteModal" width="360">
			<p slot="header" style="color:#f60;text-align:center">
				<Icon type="information-circled">
				</Icon>
				<span>
            删除确认
          </span>
			</p>
			<div style="text-align:center">
				<h2>{{this.currentField.name}} !</h2>
				<p style="margin-top: 10px">确定要删除吗？</p>
			</div>
			<div slot="footer">
				<Button type="error" size="large" long :loading="deleteLoading"
				        @click="deleteField">
					是的，删除！
				</Button>
			</div>
		</Modal>
	</Card>
</template>

<script>
  import fieldDetail from './field-detail/field-detail';
  import Group from './group/group';

  export default {
    name: 'entityDetial',
    components: {Group, fieldDetail},
    data () {
      return {
        currentEntity: {},
        // currentField: {},
        columns: [
          {
            title: 'ID',
            key: 'id',
            align: 'center',
            width: 270
          },
          {
            title: '组',
            key: 'group_name',
            align: 'center',
            width: 120
          },
          {
            title: '名称',
            key: 'name',
            align: 'center',
            width: 120
          },
          {
            title: '展示名称',
            key: 'name_cn',
            align: 'center'
          },
          {
            title: '类型',
            key: 'type',
            align: 'center',
            width: 120,
            render: (h, params) => {
              let typeDesc = '';
              let typeError = {};
              switch (params.row.type) {
                case '01':
                  typeDesc = '字段';
                  break;
                case '10':
                  typeDesc = '扩展实体';
                  break;
                case '20':
                  typeDesc = '扩展实体(明细)';
                  break;
                default:
                  typeDesc = '类型错误';
                  typeError = {style: {color: 'red'}};
                  break;
              }
              ;
              return h('div', typeError, typeDesc);
            }
          },
          {
            title: '扩展实体',
            key: 'extend_entity_id', // TODO 显示名称？
            align: 'center'
          },
          {
            title: '操作',
            key: 'action',
            width: 160,
            align: 'center',
            fixed: 'right',
            render: (h, params) => {
              return h('div', [
                h('Button', {
                  props: {
                    type: 'text',
                    size: 'small'
                  },
                  on: {
                    click: () => {
                      this.$store.commit('dm_entity_field_object', params.row);
                      this.$store.commit('dm_entity_action_view_field_detail');
                    }
                  }
                }, '查看'),
                this.editMode && this.filedEditable ? h('Button', {
                  props: {
                    type: 'text',
                    size: 'small'
                  },
                  on: {
                    click: () => {
                      this.$store.commit('dm_entity_field_object', params.row);
                      this.$store.commit('dm_entity_action_edit_field_detail');
                    }
                  }
                }, '编辑') : '',
                this.editMode && this.filedEditable ? h('Button', {
                  props: {
                    type: 'text',
                    size: 'small'
                  },
                  on: {
                    click: () => {
                      this.$store.commit('dm_entity_field_object', params.row);
                      this.delete();
                    }
                  }
                }, '删除') : ''
              ]);
            }
          }
        ],
        fields: [],
        deleteModal: false,
        deleteLoading: false,
        tableLoading: false,
        tableHeight: 300
      };
    },
    methods: {
      backward () {
        this.$store.commit('dm_entity_dispatcher_to_list');
      },
      save () {
        // check status
        let actionType = this.$store.state.dm.entity_action;
        let method;
        if (actionType === 1) { // 更新
          method = 'put';
        } else if (actionType === 2) { // 创建
          method = 'post';
        }
        this.$http({
          url: '/core/extension/defination/entities',
          method: method,
          data: this.currentEntity
        }).then(res => {
          this.$Message.success('实体创建成功！');
          this.$store.commit('dm_entity_dispatcher_to_list');
        });
      },
      create () {
        this.$store.commit('dm_entity_action_create_field_detail');
      },
      groupManage () {
        this.$store.commit('dm_entity_action_group', true);
      },
      delete () {
        this.deleteModal = true;
      },
      deleteField () {
        this.deleteLoading = true;
        // AJAX
        // TODO ajax delete
        this.deleteLoading = true;
        let delObj = this.$store.state.dm.entity_current_field_object;
        this.$http.delete(`/core/extension/defination/entities/${this.currentEntity.id}/fields/${delObj.id}`).then(res => {
          this.queryFields().then(res => {
            this.$Message.success('字段信息删除成功！');
            this.deleteModal = false;
            this.deleteLoading = false;
          });
        });
      },
      queryFields () {
        this.tableLoading = true;
        return this.$http.get(`/core/extension/defination/entities/${this.currentEntity.id}/fields`).then(res => {
          this.fields = res.data.result;
          this.tableLoading = false;
          return new Promise((resolve, reject) => {
            resolve();
          });
        }).catch(error => {
          this.$Message.error(error.msg);
        });
      }
    },
    computed: {
      filedEditable () {
        // published or banished
        return !(this.currentEntity.status === '03' || this.currentEntity.status == '04');
      },
      editMode () {
        // 编辑或者创建
        return this.$store.state.dm.entity_action === 1 || this.$store.state.dm.entity_action === 2;
      },
      currentField () {
        return this.$store.state.dm.entity_current_field_object ? this.$store.state.dm.entity_current_field_object : {};
      }
    },
    created () {
    },
    mounted () {
      this.tableHeight = this.$refs.entityDetailCard.$el.clientHeight - 300;
      this.currentEntity = Object.assign({}, this.$store.state.dm.entity_current_object);
      if(this.currentEntity && this.currentEntity.id) {
        this.queryFields();
      }
    },
    watch: {
      editMode (val) {
        if (val) {
          this.queryFields();
        }
      }
    }
  };
</script>

<style scoped>
	.entity-detail-card {
		height: 100%;
	}
</style>
