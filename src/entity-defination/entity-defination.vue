<template>

	<div class="container">
		<Row :gutter='10' class='entity-row'>
			<Col :span="24" class="entity-col">
				<!-- 列表页 -->
				<entity-table v-if="this.list_page" class="entity-table">
				</entity-table>

				<!-- 表单页 -->
				<entity-detail v-if="this.detail_page"></entity-detail>
			</Col>
		</Row>
	</div>
</template>

<script>
  import entityTable from './entity-table/entity-table';
  import entityDetail from './entity-detail/entity-detail';

  export default {
    name: 'entity-defination',
    components: {entityTable, entityDetail},
    data () {
      return {}
    },
    methods: {
    },
    computed: {
      list_page () {
        return this.$store.state.dm.entity_dispatcher === 0;
      },
      detail_page () {
        return this.$store.state.dm.entity_dispatcher === 1;
      }
    }
  };
</script>

<style scoped lang="less">
	@import "entity-defination.less";
</style>
