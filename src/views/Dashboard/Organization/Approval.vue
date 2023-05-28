<script setup lang="ts">
import { EnumValues } from 'enum-values'
import { ref, reactive, onMounted } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { organizationService } from '@/service/organization'
import { RegisterStatus, OrganizationType } from '@/utils/enums'

const route = useRoute()
const router = useRouter()

const pageSize = 10
const total = ref(0)
let currentPage = 1
let approvalList: any[]
let showList = ref(Array())

onMounted(() => {
	loadOrganizationRegisterApprovalList()
})

const loadOrganizationRegisterApprovalList = () => {
	organizationService
		.organizationRegsiterApprovalList()
		.then((res: any) => {
			console.log(res.data)
			approvalList = res.data
			total.value = approvalList.length
			showList.value = approvalList.slice((currentPage - 1) * pageSize, currentPage * pageSize)
		})
		.catch((err: any) => {})
}

const handleCurrentChange = (value: number) => {
	currentPage = value
	showList.value = approvalList.slice((currentPage - 1) * pageSize, currentPage * pageSize)
}

const indexMethod = (index: number) => {
	return (currentPage - 1) * pageSize + index + 1
}

const allow = (item: any) => {
	organizationService
		.organizationRegisterReply(item.serialNumber, EnumValues.getNameFromValue(RegisterStatus, RegisterStatus.ALLOW))
		.then((res: any) => {
			if (res.data) {
				loadOrganizationRegisterApprovalList()
			}
		})
		.catch((err: any) => {})
}

const reject = (item: any, reason: string) => {
	organizationService
		.organizationRegisterReply(
			item.serialNumber,
			EnumValues.getNameFromValue(RegisterStatus, RegisterStatus.REJECT),
			reason
		)
		.then((res: any) => {
			if (res.data) {
				loadOrganizationRegisterApprovalList()
			}
		})
		.catch((err: any) => {})
}
</script>

<template>
	<div class="w-full p-20px">
		<el-card class="w-full">
			<template #header>
				<h2 class="text-2xl">{{ route.name }}</h2>
			</template>

			<el-table :data="showList" highlight-current-row border>
				<el-table-column label="No." type="index" :index="indexMethod" width="60" />
				<el-table-column label="申请号">
					<template #default="scope">
						<CopyText :text="scope.row.serialNumber" />
					</template>
				</el-table-column>
				<el-table-column label="机构名称" prop="name" />
				<el-table-column label="机构类型">
					<template #default="scope">
						{{ OrganizationType[scope.row.type] }}
					</template>
				</el-table-column>
				<el-table-column label="状态">
					<template #default="scope">
						<ApplyStatusText :status="scope.row.status" />
					</template>
				</el-table-column>
				<el-table-column label="申请时间" prop="applyTime" />
				<el-table-column label="操作">
					<template #default="scope">
						<div class="w-full h-full flex items-center">
							<el-tooltip content="详情">
								<el-button type="primary" icon="More" circle size="default" />
							</el-tooltip>
							<AllowButton
								v-if="RegisterStatus[scope.row.status] === RegisterStatus.PROCESSED"
								title="提示"
								:text="`允许 ${scope.row.name} 的注册申请?`"
								:item="scope.row"
								@allow="allow"
							/>
							<RejectButton
								v-if="RegisterStatus[scope.row.status] === RegisterStatus.PROCESSED"
								title="提示"
								:text="`驳回 ${scope.row.name} 的注册申请?`"
								:item="scope.row"
								@reject="reject"
							/>
						</div>
					</template>
				</el-table-column>
			</el-table>

			<div class="w-full mt-20px flex items-center justify-center">
				<el-pagination
					layout="jumper, prev, pager, next,total"
					:total="total"
					:page-size="pageSize"
					@current-change="handleCurrentChange"
				/>
			</div>
		</el-card>
	</div>
</template>

<style lang="less" scoped></style>
