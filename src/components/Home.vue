<script setup>
import { computed, ref, defineExpose } from "vue";
import { Collection, Link } from "@element-plus/icons-vue";
import PageItem from "./PageItem.vue";
import { ElMessage } from "element-plus";

const props = defineProps({
  query: {
    type: String,
    default: "",
  },
});
const pages = ref([
  {
    name: "kotobahitomi",
    repoUrl: "https://github.com/diesehnsucht0/kotobahitomi",
    versions: [
      {
        date: "2025-06-03",
        log: "Project initialization",
        url: "https://diesehnsucht0.github.io/kotobahitomi/#/",
      },
    ],
  },
  {
    name: "chatto",
    repoUrl: "https://github.com/diesehnsucht0/chatto",
    versions: [
      {
        date: "2025-08-28",
        log: "Project initialization",
        url: "https://diesehnsucht0.github.io/chatto/#/",
      },
    ],
  },

  {
    name: "HCBlog",
    repoUrl: "https://github.com/diewehmut/HCBlog",
    versions: [
      {
        date: "2025-08-20",
        log: "Project initialization",
        url: "https://diewehmut.github.io/HCBlog/",
      },
    ],
  },
  {
    name: "HCPWP",
    repoUrl: "https://github.com/diewehmut/dieSWPWP",
    versions: [
      {
        version: "v1.1.1",
        date: "2025-08-24",
        log: "Optimized the page styles",
        url: "https://diewehmut.github.io/dieSWPWP1.1.1/#/",
      },
      {
        version: "v1.1.0",
        date: "2025-08-20",
        log: "Added Notes",
        url: "https://diewehmut.github.io/dieSWPWP1.1/#/",
      },
      {
        version: "v1.0.0",
        date: "2025-08-15",
        log: "Project initialization",
        url: "https://diewehmut.github.io/dieSWPWP/",
      },
    ],
  },
]);

const totalCount = computed(() =>
  pages.value.reduce((sum, p) => sum + p.versions.length, 0)
);

const normalizedQuery = computed(() => props.query.trim().toLowerCase());

function matchVersion(v, q) {
  return (
    (v.version || "").toLowerCase().includes(q) ||
    (v.log || "").toLowerCase().includes(q) ||
    (v.date || "").toLowerCase().includes(q)
  );
}

const filteredPages = computed(() => {
  const q = normalizedQuery.value;
  let result;
  if (!q) {
    result = pages.value;
  } else {
    result = pages.value
      .map((p) => ({
        ...p,
        versions: p.versions.filter(
          (v) => matchVersion(v, q) || p.name.toLowerCase().includes(q)
        ),
      }))
      .filter((p) => p.versions.length > 0);
  }

  // 按页面名称字母顺序排序
  return result.sort((a, b) =>
    a.name.toLowerCase().localeCompare(b.name.toLowerCase())
  );
});

const matchedCount = computed(() =>
  filteredPages.value.reduce((sum, p) => sum + p.versions.length, 0)
);

const activePages = ref(
  filteredPages.value.filter((p) => p.versions.length > 1).map((p) => p.name)
);

function openPage(url) {
  if (url) window.open(url, "_blank", "noopener");
}

// Expose for parent (not used now)
function openFirstResult() {
  const first = firstVersion();
  if (first) {
    window.open(first.url, "_blank", "noopener");
  } else {
    ElMessage.info("No result to open");
  }
}
function copyFirstResult() {
  const first = firstVersion();
  if (first) {
    navigator.clipboard.writeText(first.url).then(() => {
      ElMessage.success("First result link copied");
    });
  } else {
    ElMessage.info("No result to copy");
  }
}

function firstVersion() {
  const p = filteredPages.value[0];
  if (!p) return null;
  return p.versions[0] || null;
}

function formatDate(d) {
  try {
    const date = new Date(d);
    if (!isNaN(date.valueOf())) {
      const y = date.getFullYear();
      const m = date.getMonth() + 1;
      const day = date.getDate();
      return `${y}-${m}-${day}`;
    }
  } catch {}
  return d;
}

defineExpose({ openFirstResult, copyFirstResult });
</script>

<template>
  <div class="home">
    <el-card shadow="never" class="home__card">
      <template #header>
        <div class="card-header">
          <span>My Pages</span>
          <el-text size="small" type="info">
            Total {{ totalCount }} items
            <template v-if="query">
              , matched
              <el-text type="primary">{{ matchedCount }}</el-text> items
            </template>
          </el-text>
        </div>
      </template>

      <div>
        <template v-for="page in filteredPages" :key="page.name">
          <!-- 统一用 PageItem 渲染每个版本 -->
          <template v-if="page.versions.length === 1">
            <PageItem
              :page-name="page.name"
              :version="page.versions[0]"
              :repo-url="page.repoUrl"
            />
          </template>
          <el-collapse v-else v-model="activePages" class="collapse" accordion>
            <el-collapse-item :name="page.name">
              <template #title>
                <div class="page-title">
                  <el-icon><Collection /></el-icon>
                  <span>{{ page.name }}</span>
                  <el-tag size="small" effect="plain"
                    >{{ page.versions.length }} versions</el-tag
                  >
                  <a
                    class="repo-link"
                    :href="page.repoUrl"
                    target="_blank"
                    rel="noopener"
                    @click.stop
                  >
                    <el-icon class="repo-icon"><Link /></el-icon>
                    <span>Repository</span>
                  </a>
                </div>
              </template>
              <div>
                <PageItem
                  v-for="ver in page.versions"
                  :key="page.name + '@' + (ver.version || ver.date)"
                  :version="ver"
                />
              </div>
            </el-collapse-item>
          </el-collapse>
        </template>
      </div>

      <el-empty
        v-if="filteredPages.length === 0"
        description="No matching content"
      />
    </el-card>
  </div>
</template>

<style scoped>
.home {
  padding: 12px 0 24px;
}
.home__card {
  border-radius: 10px;
  border: 1px solid #eee;
}
.card-header {
  display: flex;
  align-items: baseline;
  justify-content: space-between;
}
.page-title {
  display: flex;
  align-items: center;
  gap: 8px;
  font-weight: 600;
  width: 100%;
}
.repo-link {
  color: #6ba3f5;
  text-decoration: none;
  display: flex;
  align-items: center;
  gap: 4px;
  margin-left: auto;
  font-size: 12px;
  font-weight: 500;
  transition: all 0.2s ease;
  padding: 2px 6px;
  border-radius: 4px;
}
.repo-link:hover {
  color: #4a90e2;
  background: rgba(106, 163, 245, 0.1);
}
.repo-icon {
  font-size: 12px;
}
.collapse :deep(.el-collapse-item__wrap) {
  background: transparent;
}
.single-title {
  font-weight: 600;
  font-size: 16px;
}
.single-date {
  color: #888;
  font-size: 13px;
}
.single-log {
  color: #409eff;
  margin-left: 8px;
  font-size: 13px;
}
</style>
