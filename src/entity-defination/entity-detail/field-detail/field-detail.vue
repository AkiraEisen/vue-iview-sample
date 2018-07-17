<template>
	<Modal v-model="modalShow" width="800" height="200" title='字段详情'>
		<Form :label-width="85" :model="currentField">
			<Row>
				<Col span="12">
					<FormItem label="实体ID" prop="entity_id">
						<Input size="small" type="text" clearable v-model="currentField.entity_id" :disabled="true"></Input>
					</FormItem>
				</Col>
				<Col span="12">
					<FormItem label="字段ID" prop="id">
						<Input size="small" type="text" clearable v-model="currentField.id" :disabled="true"></Input>
					</FormItem>
				</Col>
			</Row>
			<Row>
				<Col span="12">
					<FormItem label="字段名称" prop="name">
						<Input size="small" type="text" clearable v-model="currentField.name" :disabled="!editMode"></Input>
					</FormItem>
				</Col>
				<Col span="12">
					<FormItem label="展示名称" prop="name_cn">
						<Input size="small" type="text" clearable v-model="currentField.name_cn" :disabled="!editMode"></Input>
					</FormItem>
				</Col>
			</Row>
			<Row>
				<Col span="12">
					<FormItem label="类型" prop="type">
						<Select size="small" v-model="currentField.type" :disabled="!editMode">
							<Option value="01">
								字段
							</Option>
							<Option value="10">
								扩展实体
							</Option>
							<Option value="20">
								扩展实体(明细)
							</Option>
						</Select>
					</FormItem>
				</Col>
				<Col span="12">
					<FormItem label="扩展实体" prop="extend_entity_id" v-show="currentField.type === '10' || currentField.type === '20'">
						<Input size="small" type="text" clearable v-model="currentField.extend_entity_id"
						       :disabled="!editMode"></Input>
					</FormItem>
				</Col>
				<Col span="6">
					<FormItem label="数据类型" prop="data_type" v-show="currentField.type === '01'">
						<Select size="small" v-model="currentField.data_type" :disabled="!editMode">
							<Option value="01">
								varchar
							</Option>
							<Option value="02">
								integer
							</Option>
							<Option value="03">
								boolean
							</Option>
							<Option value="04">
								float
							</Option>
							<Option value="05">
								timestamp
							</Option>
						</Select>
					</FormItem>
				</Col>
				<Col span="6">
					<FormItem label="数据长度" prop="data_length"  v-show="currentField.type === '01' && currentField.data_type === '01'">
						<InputNumber size="small" type="text" clearable v-model="currentField.data_length" :disabled="!editMode"></InputNumber>
					</FormItem>
				</Col>
				<Col span="6">
					<FormItem label="自增值" prop="auto_increment"  v-show="currentField.type === '01'  && currentField.data_type === '02'">
						<InputNumber  size="small" type="text" clearable v-model="currentField.auto_increment" :disabled="!editMode"></InputNumber>
					</FormItem>
				</Col>
			</Row>
			<!--<Row>-->

			<!--</Row>-->
			<!--<Tabs>-->
				<!--<TabPane label="存储属性" class="tab-panel">-->
					<!--<Row>-->
						<!--<Col span="6">-->
							<!--<FormItem label="数据类型" prop="data_type">-->
								<!--<Select size="small" v-model="currentField.data_type" :disabled="!editMode">-->
									<!--<Option value="01">-->
										<!--varchar-->
									<!--</Option>-->
									<!--<Option value="02">-->
										<!--integer-->
									<!--</Option>-->
									<!--<Option value="03">-->
										<!--boolean-->
									<!--</Option>-->
									<!--<Option value="04">-->
										<!--float-->
									<!--</Option>-->
									<!--<Option value="05">-->
										<!--timestamp-->
									<!--</Option>-->
								<!--</Select>-->
							<!--</FormItem>-->
						<!--</Col>-->
						<!--<Col span="6">-->
							<!--<FormItem label="数据长度" prop="data_length" v-show="currentField.data_type === '01'">-->
								<!--<InputNumber size="small" type="text" clearable v-model="currentField.data_length" :disabled="!editMode"></InputNumber>-->
							<!--</FormItem>-->
						<!--</Col>-->
						<!--<Col span="6">-->
							<!--<FormItem label="自增值" prop="auto_increment" v-show="currentField.data_type === '02'">-->
								<!--<InputNumber  size="small" type="text" clearable v-model="currentField.auto_increment" :disabled="!editMode"></InputNumber>-->
							<!--</FormItem>-->
						<!--</Col>-->
					<!--</Row>-->
				<!--</TabPane>-->
				<!--<TabPane label="渲染属性" class="tab-panel">-->
					<!--<Row>-->
						<!--<Col span="12">-->
							<!--<FormItem label="组件类型" prop="component_type">-->
								<!--<Select size="small" v-model="currentField.component_type" :disabled="!editMode">-->
									<!--<Option value="01">-->
										<!--Input-->
									<!--</Option>-->
									<!--<Option value="02">-->
										<!--Select-->
									<!--</Option>-->
									<!--<Option value="03">-->
										<!--TextArea-->
									<!--</Option>-->
									<!--<Option value="04">-->
										<!--RadioButton-->
									<!--</Option>-->
									<!--<Option value="05">-->
										<!--Switch-->
									<!--</Option>-->
									<!--<Option value="06">-->
										<!--DatePicker-->
									<!--</Option>-->
								<!--</Select>-->
							<!--</FormItem>-->
						<!--</Col>-->
						<!--<Col span="12">-->
							<!--<FormItem label="组ID" prop="group_id">-->
								<!--&lt;!&ndash;<Input size="small" type="text" clearable v-model="currentField.group_id" :disabled="!editMode"></Input>&ndash;&gt;-->
								<!--<Select size="small" v-model="currentField.group_id" :disabled="!editMode">-->
									<!--<Option :value="group.key" v-for="group in this.groups" :key="group.key">-->
										<!--{{group.val}}-->
									<!--</Option>-->
								<!--</Select>-->
							<!--</FormItem>-->
						<!--</Col>-->
					<!--</Row>-->

					<!--<Row>-->
						<!--<Col span="12">-->
							<!--<FormItem label="是否标题" prop="is_title">-->
								<!--<i-switch v-model="currentField.is_title" :disabled="!editMode">-->
									<!--<span slot="open">是</span>-->
									<!--<span slot="close">否</span>-->
								<!--</i-switch>-->
							<!--</FormItem>-->
						<!--</Col>-->
						<!--<Col span="12">-->
							<!--<FormItem label="国际化" prop="international_key">-->
								<!--<Input size="small" type="text" clearable v-model="currentField.international_key"-->
								       <!--:disabled="!editMode"></Input>-->
							<!--</FormItem>-->
						<!--</Col>-->
					<!--</Row>-->

					<!--<Row>-->
						<!--<Col span="12">-->
							<!--<FormItem label="Y" prop="position_row">-->
								<!--<Input size="small" type="text" clearable v-model="currentField.position_row" :disabled="!editMode"></Input>-->
							<!--</FormItem>-->
						<!--</Col>-->
						<!--<Col span="12">-->
							<!--<FormItem label="X" prop="position_col">-->
								<!--<Input size="small" type="text" clearable v-model="currentField.position_col" :disabled="!editMode"></Input>-->
							<!--</FormItem>-->
						<!--</Col>-->
					<!--</Row>-->

					<!--<Row>-->
						<!--<Col span="12">-->
							<!--<FormItem label="跨列数" prop="col_span">-->
								<!--<InputNumber size="small" type="text"-->
								             <!--:max="4" :min="1" clearable v-model="currentField.col_span"-->
								             <!--:disabled="!editMode"></InputNumber>-->
							<!--</FormItem>-->
						<!--</Col>-->
						<!--<Col span="12">-->
							<!--<FormItem label="是否列表项" prop="is_table_header_field">-->
								<!--<i-switch v-model="currentField.is_table_header_field" :disabled="!editMode">-->
									<!--<span slot="open">是</span>-->
									<!--<span slot="close">否</span>-->
								<!--</i-switch>-->
							<!--</FormItem>-->
						<!--</Col>-->
					<!--</Row>-->
					<!--<Row>-->
						<!--<Col span="12">-->
							<!--<FormItem label="是否查询键" prop="is_search_key">-->
								<!--<i-switch v-model="currentField.is_search_key" :disabled="!editMode">-->
									<!--<span slot="open">是</span>-->
									<!--<span slot="close">否</span>-->
								<!--</i-switch>-->
							<!--</FormItem>-->
						<!--</Col>-->
					<!--</Row>-->
					<!--<Row>-->
						<!--<Col span="24">-->
							<!--&lt;!&ndash; TODO 允许用户写SQL&ndash;&gt;-->
							<!--<FormItem label="查询策略" prop="search_category">-->
								<!--<Input size="small" type="textarea" :rows="4" clearable v-model="currentField.search_category"-->
								       <!--:disabled="!editMode"></Input>-->
							<!--</FormItem>-->
						<!--</Col>-->
					<!--</Row>-->
				<!--</TabPane>-->
			<!--</Tabs>-->


		</Form>
		<div slot="footer">
			<Button @click="this.cancelHandler" size="small" icon="close-round">取消</Button>
			<Button type="success" @click="this.okHandler" size="small" icon="checkmark-round">确定</Button>
		</div>
	</Modal>
</template>

<script>
  export default {
    name: 'fieldDetail',
    data () {
      return {
        groups: [],
        currentField: {}
      };
    },
    methods: {
      okHandler () {
        if (this.editMode) {
          let actionType = this.$store.state.dm.entity_action_field_detail;
          let method;
          if (actionType === 2) { // update
            method = 'put';
          } else if (actionType === 3) { // create
            method = 'post';
          }
          this.$http({
            url: '/core/extension/defination/fields',
            method: method,
            data: this.currentField
          }).then(res => {
            this.$Message.success(actionType === 3 ? '实体创建成功！' : '实体更新成功！');
            // 更新列表
            this.$emit('refresh-fields');
            // 关闭POPUP
            this.$store.commit('dm_entity_action_hide_field_detail');
          });
        } else {
          // do nothing
          this.$store.commit('dm_entity_action_hide_field_detail');
        }
      },
      cancelHandler () {
        this.$store.commit('dm_entity_action_hide_field_detail');
      }
    },
    computed: {
      modalShow: {
        get () {
          return this.$store.state.dm.entity_action_field_detail !== 0;
        },
        set () {
          this.$store.commit('dm_entity_action_hide_field_detail');
        }
      },
      modalState () {
        return this.$store.state.dm.entity_action_field_detail !== 0;
      },
      editMode () {
        return (this.$store.state.dm.entity_action_field_detail === 2 || this.$store.state.dm.entity_action_field_detail === 3);
      }
    },
    watch: {
      modalState: function (val) {
        if (val) {
          // 初始化
          let entity = this.$store.state.dm.entity_current_object;
          this.groups = [];
          this.$http.get(`/core/extension/defination/entities/${entity.id}/formgroups`).then(res => {
            // 更新组下拉
            res.data.result.forEach(item => {
              this.groups.push({key: item.id, val: item.name_cn});
            });
            if (this.$store.state.dm.entity_action_field_detail === 3) {
              // 新建时，清空当前属性，并将实体ID赋值。
              this.currentField = {};
              this.currentField.entity_id = entity.id;
              this.currentField.is_title = false;
              this.currentField.is_search_key = false;
              this.currentField.is_table_header_field = false;
            } else {
              this.currentField = Object.assign({}, this.$store.state.dm.entity_current_field_object);
            }
          });
        }
      }
    }
  };
</script>

<style scoped lang="less">
  .tab-panel {
	  height: 300px;
	  overflow-y: auto;
  }
</style>
