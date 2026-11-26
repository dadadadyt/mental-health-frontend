<template>
    <div class="emotionDiary-container">
        <div class="header-section">
            <div class="header-content">
                <el-image :src="iconUrl" style="width: 60px;height: 60px"></el-image>
                <h1>情绪日志</h1>
            </div>
        </div>
        <div class="content">
            <!-- 情绪评分 -->
            <div class="diary-card">
                <div class="title">今日情绪评分</div>
                <div class="section">
                    <p>您今天的整体情绪状态如何？(1-10分)</p>
                    <div class="rate">
                        <el-rate 
                            v-model="diaryForm.moodScore"
                            :texts="emotionStatus"
                            show-texts
                            :max="10"
                            size="large"
                        />
                    </div>
                </div>
            </div>
            <!-- 主要情绪 -->
            <div class="diary-card">
                <div class="title">主要情绪</div>
                <div class="emotion-grid">
                    <div v-for="emotion in emotionOptions" :key="emotion.name" class="emotion-card" :class="{'selected': emotion.name === diaryForm.dominantEmotion}" @click="selectEmotion(emotion.name)">
                        <el-image :src="emotion.url" style="width: 50px;height: 50px"></el-image>
                        <div class="emotion-name">{{emotion.name}}</div>
                    </div>
                </div>
            </div>
            <!-- 详细记录 -->
            <div class="diary-card">
                <div class="title">详细记录</div>
                <div class="detail-form">
                    <div class="form-group">
                        <div class="form-label">情绪触发因素</div>
                        <el-input v-model="diaryForm.emotionTriggers" placeholder="今天什么事情影响了您的情绪？" type="textarea" :rows="3" maxLength="1000" show-word-limit></el-input>
                    </div>
                     <div class="form-group">
                        <div class="form-label">今日感想</div>
                        <el-input v-model="diaryForm.diaryContent" placeholder="写下您今天的想法、感受或发生的有趣事情..." type="textarea" :rows="5" maxLength="2000" show-word-limit></el-input>
                    </div>
                    <!-- 生活指标 -->
                    <div class="life-indicators">
                        <div class="indicator-group">
                            <div class="form-label">睡眠质量</div>
                           <el-select v-model="diaryForm.sleepQuality" placeholder="请选择">
                                <el-option label="很差" :value="1"></el-option>
                                <el-option label="较差" :value="2"></el-option>
                                <el-option label="一般" :value="3"></el-option>
                                <el-option label="良好" :value="4"></el-option>
                                <el-option label="优秀" :value="5"></el-option>
                            </el-select>
                        </div>
                        <div class="indicator-group">
                            <div class="form-label">压力水平</div>
                            <el-select v-model="diaryForm.stressLevel" placeholder="请选择">
                                <el-option label="很低" :value="1"></el-option>
                                <el-option label="较低" :value="2"></el-option>
                                <el-option label="中等" :value="3"></el-option>
                                <el-option label="较高" :value="4"></el-option>
                                <el-option label="很高" :value="5"></el-option>
                            </el-select>
                        </div>
                    </div>
                    <div class="action-buttons">
                        <el-button  @click="resetForm">重置</el-button>
                        <el-button type="primary" @click="submit">提交记录</el-button>
                    </div>
                </div>
            </div>
            <!-- 我的记录 -->
            <div class="diary-card">
                <div class="title">我的记录</div>
                <div class="filter-bar">
                    <el-select v-model="query.dominantEmotion" placeholder="全部情绪" clearable style="width: 140px">
                        <el-option v-for="emotion in emotionOptions" :key="emotion.name" :label="emotion.name" :value="emotion.name"></el-option>
                    </el-select>
                    <el-date-picker
                        v-model="query.dateRange"
                        type="daterange"
                        value-format="YYYY-MM-DD"
                        range-separator="至"
                        start-placeholder="开始日期"
                        end-placeholder="结束日期"
                        style="width: 260px"
                    />
                    <el-input v-model="query.keyword" placeholder="搜索日记内容" clearable style="width: 200px" />
                    <el-button type="primary" @click="searchList">查询</el-button>
                </div>
                <el-table :data="diaryList" v-loading="loading" style="width: 100%">
                    <el-table-column prop="diaryDate" label="记录日期" width="120" />
                    <el-table-column prop="dominantEmotion" label="主要情绪" width="100" />
                    <el-table-column label="情绪评分" width="160">
                        <template #default="scope">
                            <el-rate :model-value="scope.row.moodScore" :max="10" disabled />
                        </template>
                    </el-table-column>
                    <el-table-column prop="diaryContent" label="今日感想" show-overflow-tooltip />
                    <el-table-column prop="sleepQuality" label="睡眠" width="80" />
                    <el-table-column prop="stressLevel" label="压力" width="80" />
                </el-table>
                <el-pagination
                    v-model:current-page="query.pageNum"
                    v-model:page-size="query.pageSize"
                    :total="total"
                    :page-sizes="[5, 10, 20]"
                    layout="total, sizes, prev, pager, next"
                    style="margin-top: 16px; justify-content: flex-end"
                    @current-change="loadList"
                    @size-change="searchList"
                />
            </div>
        </div>
    </div>
</template>
<script setup>
    import { dayjs, ElMessage } from 'element-plus'
    import { ref, reactive } from 'vue'
    import { addEmotionDiary, getEmotionDiaryPage } from '@/api/frontend'

    // 情绪评分
    const emotionStatus = ['绝望崩溃', '消沉抑郁', '焦虑烦躁', '低落不悦', '平静淡然', '轻松惬意', '愉悦舒心', '欢欣满足', '兴奋欣喜', '极致幸福']

    // 情绪选项
    const emotionOptions = [
        { name: '开心', url: new URL('@/assets/images/开心.png', import.meta.url).href },
        { name: '平静', url: new URL('@/assets/images/平静.png', import.meta.url).href },
        { name: '焦虑', url: new URL('@/assets/images/焦虑.png', import.meta.url).href },
        { name: '悲伤', url: new URL('@/assets/images/悲伤.png', import.meta.url).href },
        { name: '兴奋', url: new URL('@/assets/images/兴奋.png', import.meta.url).href },
        { name: '疲惫', url: new URL('@/assets/images/疲惫.png', import.meta.url).href },
        { name: '惊讶', url: new URL('@/assets/images/惊讶.png', import.meta.url).href },
        { name: '困惑', url: new URL('@/assets/images/困惑.png', import.meta.url).href },
    ]

    const selectEmotion = (emotion) => {
        diaryForm.dominantEmotion = emotion
    }

    const diaryForm = reactive({    
        diaryDate: dayjs().format('YYYY-MM-DD'),
        moodScore: null,
        dominantEmotion: '',
        emotionTriggers: '',
        diaryContent: '',
        sleepQuality: null,
        stressLevel: null
    })

    const resetForm = () => {
        Object.assign(diaryForm, {
            diaryDate: dayjs().format('YYYY-MM-DD'),
            moodScore: null,
            dominantEmotion: '',
            emotionTriggers: '',
            diaryContent: '',
            sleepQuality: null,
            stressLevel: null
        })
    }

    const submit = () => {
        console.log(diaryForm)
        if (!diaryForm.moodScore) {
            ElMessage.error('请选择情绪评分')
            return
        }
        addEmotionDiary(diaryForm).then(() => {
            ElMessage.success('提交成功')
            resetForm()
            searchList()
        })
    }

    const diaryList = ref([])
    const total = ref(0)
    const loading = ref(false)
    const query = reactive({
        pageNum: 1,
        pageSize: 10,
        dominantEmotion: '',
        keyword: '',
        dateRange: []
    })

    const loadList = () => {
        loading.value = true
        getEmotionDiaryPage({
            pageNum: query.pageNum,
            pageSize: query.pageSize,
            dominantEmotion: query.dominantEmotion || undefined,
            keyword: query.keyword || undefined,
            startDate: query.dateRange && query.dateRange[0] ? query.dateRange[0] : undefined,
            endDate: query.dateRange && query.dateRange[1] ? query.dateRange[1] : undefined
        }).then(res => {
            diaryList.value = res.records || []
            total.value = res.total || 0
        }).finally(() => {
            loading.value = false
        })
    }

    const searchList = () => {
        query.pageNum = 1
        loadList()
    }

    loadList()

    const iconUrl = new URL('@/assets/images/like.png', import.meta.url).href
</script>
<style lang="scss" scoped>
    .emotionDiary-container {
    background: linear-gradient(135deg, #fafbfc 0%, #f7f9fc 50%, #f2f6fa 100%);
    .header-section {
        background: linear-gradient(135deg, #7ED321 0%, #F5A623 100%);
        color: white;
        padding: 48px;
        .header-content {
            display: flex;
            align-items: center;
            gap: 12px;
        }
    }
    .content {
        margin: 0 auto;
        width: 980px;
        padding: 20px;
        .diary-card {
            margin-bottom: 20px;
            background: white;
            border-radius: 10px;
            padding: 20px;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.05);
            .filter-bar {
                display: flex;
                flex-wrap: wrap;
                gap: 12px;
                margin-bottom: 16px;
            }
            .title {
                margin-bottom: 20px;
                font-size: 25px;
                font-weight: 600;
                color: #374151;
            }
            .section {
                margin-bottom: 20px;
                p {
                    font-size: 15px;
                    color: #6B7280;
                    margin-bottom: 15px;
                }
            }
            .emotion-grid {
                display: flex;
                flex-wrap: wrap;
                gap: 10px;
                .emotion-card {
                    padding: 15px;
                    border: 2px solid #E5E7EB;
                    border-radius: 15px;
                    text-align: center;
                    cursor: pointer;
                    background: #F9FAFB;
                    .emotion-name {
                        margin-top: 10px;
                        padding: 0 75px;
                        color: #374151;
                    }
                    &.selected {
                        border-color: #7ED321;
                        background: #F0FDF4;
                        transform: translateY(-3px);
                    }
                }
            }
            .detail-form {
                .form-label {
                    margin: 10px 0;
                    color: #374151;
                }
                .life-indicators {
                    display: flex;
                    gap: 20px;
                    .indicator-group {
                        flex: 1;
                    }
                }
                .action-buttons {
                    margin-top: 40px
                }
            }
        }
    }
}
</style>
