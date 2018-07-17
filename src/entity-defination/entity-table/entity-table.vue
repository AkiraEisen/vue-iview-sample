<template>
    <Card class="entity-table-card" ref="entityTableCard">
        <span slot="title"><h2>实体定义</h2></span>
        <span slot="extra"><Button type="primary" size='small' icon="plus-round" @click="this.create">新增</Button></span>
        <span>
			<Form :label-width="85">
				<Row>
					<Col span="8">
						<FormItem label="实体名称">
							<Input size="small" type="text" clearable v-model="queryConditions.entity_name"></Input>
						</FormItem>
					</Col>
					<Col span="1">
						<FormItem>
							<Button size='small' icon="ios-search" type="primary" @click="queryEntity">搜索</Button>
						</FormItem>
					</Col>
				</Row>
			</Form>
		</span>
        <Table
                size="small"
                :border="true"
                :stripe="true"
                :loading="this.tableLoading"
                :columns="this.columns"
                :data="this.entity_current_collection"
                :height="tableHeight"></Table>
        <div class="pagenationContainer">
            <Page :total="pageTotal" class="pagenation"
                  :current="pageNum"
                  size="small"
                  :page-size="pageSize"
                  show-elevator show-sizer show-total placement="top"
                  @on-change="pageChangeHandler"
                  @on-page-size-change="pageSizeChangeHanlder"></Page>
        </div>
        <Modal v-model="deleteModal" width="360">
            <p slot="header" style="color:#f60;text-align:center">
                <Icon type="information-circled">
                </Icon>
                <span>
            删除确认
          </span>
            </p>
            <div style="text-align:center">
                <h2>{{this.entity_current_object.name}} !</h2>
                <p style="margin-top: 10px">确定要删除吗？</p>
            </div>
            <div slot="footer">
                <Button type="error" size="large" long :loading="deleteLoading"
                        @click="deleteEntity">
                    是的，删除！
                </Button>
            </div>
        </Modal>
    </Card>
</template>

<script>
  export default {
    name: 'entityTable',
    data () {
      return {
        // table height
        tableHeight: 400,
        // entity table
        columns: [
          {
            title: 'ID',
            key: 'id',
            align: 'center',
            width: 275
          },
          {
            title: '类型',
            key: 'type', // 00:实体 10:扩展实体 11:扩展实体（明细）
            align: 'center',
            render: (h, params) => {
              let typeDesc = '';
              let typeError = {};
              switch (params.row.type) {
                case '00':
                  typeDesc = '实体';
                  break;
                case '10':
                  typeDesc = '扩展实体';
                  break;
                case '11':
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
            title: '名称',
            key: 'name',
            align: 'center'
          },
          {
            title: '展示名称',
            key: 'name_cn',
            align: 'center'
          },
          // TODO 移至页面定义
          // {
          //   title: '分页',
          //   key: 'pagenation', // true:是 false:否
          //   // width: '100%',
          //   align: 'center'
          // },
          {
            title: '连接ID',
            key: 'conn_id',
            align: 'center'
          },
          {
            title: '状态',
            key: 'status', // TODO 00:创建 01:保存 02:同步 03:发布 04:废弃
            // width: '100%',
            align: 'center',
            render: (h, params) => {
              let typeDesc = '';
              let typeError = {};
              switch (params.row.status) {
                // case '00':
                //   typeDesc = '创建';
                //   break;
                case '01':
                  typeDesc = '编辑';
                  break;
                // case '02':
                //   typeDesc = '同步';
                //   break;
                case '03':
                  typeDesc = '发布';
                  break;
                case '04':
                  typeDesc = '废弃';
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
            title: '描述',
            key: 'description',
            // width: '100%',
            align: 'center'
          },
          {
            title: '操作',
            key: 'action',
            width: 300,
            align: 'center',
            fixed: 'right',
            render: (h, params) => {
              return h('div', [
                h('Button', {
                  props: {
                    type: 'text',
                    size: 'small'
                  },
                  directives: [
                    {
                      name: 'clipboard',
                      rawName: 'v-clipboard:copy',
                      value: (params.row.id),
                      expression: 'param.row.id',
                      arg: 'copy'
                    },
                    {
                      name: 'clipboard',
                      rawName: 'v-clipboard:success',
                      value: (this.handlerClipboardSuccess),
                      expression: 'this.copyHandler',
                      arg: 'success'
                    }
                  ]
                }, '复制ID'),
                h('Button', {
                  props: {
                    type: 'text',
                    size: 'small'
                  },
                  on: {
                    click: () => {
                      this.$store.commit('dm_entity_object', {id: params.row.id});
                      this.view();
                    }
                  }
                }, '查看'),
                h('Button', {
                  props: {
                    type: 'text',
                    size: 'small'
                  },
                  on: {
                    click: () => {
                      this.$store.commit('dm_entity_object', {id: params.row.id});
                      this.edit();
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
                      this.$store.commit('dm_entity_object', {id: params.row.id});
                      this.delete();
                    }
                  }
                }, '删除'),
                h('Button', {
                  props: {
                    type: 'text',
                    size: 'small'
                  },
                  on: {
                    click: () => {
                      this.sync(params.row.id);
                    }
                  }
                }, '同步'),
                h('Button', {
                  props: {
                    type: 'text',
                    size: 'small'
                  },
                  on: {
                    click: () => {
                      this.publish(params.row);
                    }
                  }
                }, params.row.status === '03' ? '撤销' : '发布')
              ]);
            }
          }
        ],
        pageNum: 1,
        pageSize: 20,
        pageTotal: 1,
        queryConditions: {},
        tableLoading: false,
        // delete modal
        deleteModal: false,
        deleteLoading: false
      };
    },
    methods: {
      handlerClipboardSuccess (arg) {
        this.$Message.success(`ID{${arg.text}}已复制到剪切板!`);
      },
      queryEntity () {
        // 请求第一页数据
        this.pageChangeHandler(1);
      },
      queryEntityTableData () {
        // this.tableLoading = true;
        this.tableLoading = true;
        return this.$http.get('/core/extension/defination/entities', {
          params: {
            pagenum: this.pageNum,
            pagesize: this.pageSize,
            entityname: this.queryConditions.entity_name
          }
        }).then(res => {
          this.pageTotal = res.data.total;
          this.$store.commit('dm_entity_collection', res.data.result.data);
          this.tableLoading = false;
          return new Promise((resolve, reject) => {
            resolve();
          });
        });
      },
      pageChangeHandler (pageIndex) {
        // 重新请求数据
        this.pageNum = pageIndex;
        this.queryEntityTableData();
      },
      pageSizeChangeHanlder () {
        this.pageChangeHandler(1);
      },
      view () {
        this.$store.commit('dm_entity_dispatcher_to_detail');
        this.$store.commit('dm_entity_action_view');
      },
      edit () {
        this.$store.commit('dm_entity_dispatcher_to_detail');
        this.$store.commit('dm_entity_action_edit');
      },
      delete () {
        this.deleteModal = true;
      },
      create () {
        // TODO get id
        this.$store.commit('dm_entity_dispatcher_to_detail');
        this.$store.commit('dm_entity_action_create');
        this.$store.commit('dm_entity_object', {status: '01'});
      },
      deleteEntity () {
        // TODO ajax delete
        this.deleteLoading = true;
        let deleteObject = this.$store.state.dm.entity_current_object;
        this.$http.delete(`/core/extension/defination/entities/${deleteObject.id}`).then(res => {
          this.queryEntityTableData().then(res => {
            this.$Message.success('删除成功！');
            this.deleteModal = false;
            this.deleteLoading = false;
          });
        });
      },
      sync (entityID) {
        console.log(entityID);
      },
      publish (row) {
        let statusData;
        let msg;
        if (row.status === '03') {
          statusData = {id: row.id, status: '01'};
          msg = '撤销';
        } else {
          statusData = {id: row.id, status: '03'};
          msg = '发布';
        }
        // TODO @师小壮 该实体是否被引用，如被引用，则可撤销，否则不可。 @何永泉 如何判定引用
        this.$http({url: '/core/extension/defination/entities', method: 'put', data: statusData})
          .then(() => {
            this.queryEntityTableData();
            this.$Message.success(`${msg}成功！`);
          })
          .catch(() => {
            this.$Message.error(`不可${msg}！`);
          });
      }
    },
    computed: {
      entity_current_collection () {
        return this.$store.state.dm.entity_current_collection ? this.$store.state.dm.entity_current_collection : [];
      },
      entity_current_object () {
        return this.$store.state.dm.entity_current_object ? this.$store.state.dm.entity_current_object : {};
      }
    },
    mounted () {
      this.tableHeight = this.$refs.entityTableCard.$el.clientHeight - 165;
      this.queryEntityTableData();
    },
    created () {
    }
  };
</script>

<style scoped>
    .entity-table-card {
        height: 100%;
    }

    .pagenation {
        margin-top: 5px;
    }

    .pagenationContainer {
        text-align: center;
    }
</style>
