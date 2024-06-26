<script setup>
import useClipboard from 'vue-clipboard3'
import { ref, h, computed, onMounted } from 'vue'
import { useI18n } from 'vue-i18n'
import { useRoute, useRouter } from 'vue-router'
import { useIsMobile } from '../utils/composables'
import { DarkModeFilled, LightModeFilled, MenuFilled, AdminPanelSettingsFilled, SendFilled } from '@vicons/material'
import { GithubAlt, Language, User, Home, Copy } from '@vicons/fa'
import AdminContact from './admin/AdminContact.vue'

import { useGlobalState } from '../store'
import { api } from '../api'
const { toClipboard } = useClipboard()
const message = useMessage()

const {
    jwt, localeCache, toggleDark, isDark,
    showAuth, adminAuth, auth, loading
} = useGlobalState()
const { showLogin, openSettings, settings } = useGlobalState()
const route = useRoute()
const router = useRouter()
const isMobile = useIsMobile()
const isAdminRoute = computed(() => route.path.includes('admin'))

const showMobileMenu = ref(false)
const showNewEmail = ref(false)
const showLogout = ref(false)
const showDelteAccount = ref(false)
const password = ref('')
const showPassword = ref(false)
const emailName = ref("")
const emailDomain = ref("")

const login = async () => {
    try {
        jwt.value = password.value;
        await api.getSettings()
        location.reload()
    } catch (error) {
        message.error(error.message || "error");
    }
}

const logout = () => {
    jwt.value = '';
    location.reload()
}

const authFunc = async () => {
    try {
        location.reload()
    } catch (error) {
        message.error(error.message || "error");
    }
}

const changeLocale = (locale) => {
    localeCache.value = locale;
    location.reload()
}

const { t } = useI18n({
    locale: localeCache.value || 'zh',
    messages: {
        en: {
            title: 'Cloudflare Temp Email',
            dark: 'Dark',
            light: 'Light',
            login: 'Login',
            logout: 'Logout',
            logoutConfirm: 'Are you sure to logout?',
            delteAccount: "Delete Account",
            delteAccountConfirm: "Are you sure to delete your account and all emails for this account?",
            accessHeader: 'Access Password',
            accessTip: 'Please enter the correct password',
            settings: 'Settings',
            home: 'Home',
            menu: 'Menu',
            user: 'User',
            sendbox: 'Send Box',
            sendMail: 'Send Mail',
            pleaseGetNewEmail: 'Please login or click "Get New Email" button to get a new email address',
            getNewEmail: 'Get New Email',
            getNewEmailTip1: 'Please input the email you want to use. only allow ., a-z, A-Z and 0-9',
            getNewEmailTip2: 'Levaing it blank will generate a random email address.',
            getNewEmailTip3: 'You can choose a domain from the dropdown list.',
            yourAddress: 'Your email address is',
            password: 'Password',
            passwordTip: 'Please copy the password and you can use it to login to your email account.',
            cancel: 'Cancel',
            ok: 'OK',
            copy: 'Copy',
            copied: 'Copied',
            showPassword: 'Show Password',
            fetchAddressError: 'Login password is invalid or account not exist, it may be network connection issue, please try again later.',
            mailV1Alert: 'You have some mails in v1, please click here to login and visit your history mails.',
            generateName: 'Generate Fake Name',
        },
        zh: {
            title: 'Cloudflare 临时邮件',
            dark: '暗色',
            light: '亮色',
            login: '登录',
            logout: '登出',
            logoutConfirm: '确定要登出吗？',
            delteAccount: "删除账户",
            delteAccountConfirm: "确定要删除你的账户和其中的所有邮件吗?",
            accessHeader: '访问密码',
            accessTip: '请输入站点访问密码',
            settings: '设置',
            home: '主页',
            menu: '菜单',
            user: '用户',
            sendbox: '发件箱',
            sendMail: '发送邮件',
            pleaseGetNewEmail: '请"登录"或点击 "获取新邮箱" 按钮来获取一个新的邮箱地址',
            getNewEmail: '获取新邮箱',
            getNewEmailTip1: '请输入你想要使用的邮箱地址, 只允许 ., a-z, A-Z, 0-9',
            getNewEmailTip2: '留空将会生成一个随机的邮箱地址。',
            getNewEmailTip3: '你可以从下拉列表中选择一个域名。',
            yourAddress: '你的邮箱地址是',
            password: '密码',
            passwordTip: '请复制密码，你可以使用它登录你的邮箱。',
            cancel: '取消',
            ok: '确定',
            copy: '复制',
            copied: '已复制',
            showPassword: '查看密码',
            fetchAddressError: '登录密码无效或账号不存在，也可能是网络连接异常，请稍后再尝试。',
            mailV1Alert: '你有一些 v1 版本的邮件，请点击此处登录查看。',
            generateName: '生成随机名字',
        }
    }
});

const showUserMenu = computed(() => !!settings.value.address)

const menuOptions = computed(() => [
    {
        label: () => h(
            NButton,
            {
                text: true,
                size: "small",
                style: "width: 100%",
                onClick: () => { router.push('/'); showMobileMenu.value = false; }
            },
            {
                default: () => t('home'),
                icon: () => h(NIcon, { component: Home })
            }
        ),
        key: "home"
    },
    {
        label: () => h(
            NButton,
            {
                text: true,
                size: "small",
                style: "width: 100%",
                onClick: () => { router.push('/admin'); showMobileMenu.value = false; }
            },
            {
                default: () => "Admin",
                icon: () => h(NIcon, { component: AdminPanelSettingsFilled }),
            }
        ),
        show: !!adminAuth.value,
        key: "admin"
    },
    {
        label: () => h(
            NButton,
            {
                text: true,
                size: "small",
                style: "width: 100%",
            },
            {
                default: () => t('user'),
                icon: () => h(NIcon, { component: User }),
            }
        ),
        show: showUserMenu.value,
        key: "user",
        children: [
            {
                label: () => h(
                    NButton,
                    {
                        text: true,
                        size: "small",
                        style: "width: 100%",
                        onClick: () => { router.push('/sendbox'); showMobileMenu.value = false; }
                    },
                    { default: () => t('sendbox') }
                ),
                key: "sendbox"
            },
            {
                label: () => h(
                    NButton,
                    {
                        text: true,
                        size: "small",
                        style: "width: 100%",
                        onClick: () => { showPassword.value = true; showMobileMenu.value = false; }
                    },
                    { default: () => t('showPassword') }
                ),
                key: "showPassword"
            },
            {
                label: () => h(
                    NButton,
                    {
                        text: true,
                        size: "small",
                        style: "width: 100%",
                        onClick: () => { router.push('/settings'); showMobileMenu.value = false; }
                    },
                    { default: () => t('settings') }
                ),
                show: openSettings.value.enableAutoReply,
                key: "settings"
            },
            {
                label: () => h(
                    NButton,
                    {
                        text: true,
                        size: "small",
                        style: "width: 100%",
                        onClick: () => { showLogout.value = true; showMobileMenu.value = false; }
                    },
                    { default: () => t('logout') }
                ),
                key: "logout"
            },
            {
                label: () => h(
                    NButton,
                    {
                        text: true,
                        size: "small",
                        style: "width: 100%",
                        onClick: () => { showDelteAccount.value = true; showMobileMenu.value = false; }
                    },
                    { default: () => t('delteAccount') }
                ),
                show: openSettings.value.enableUserDeleteEmail,
                key: "delte_account"
            }
        ]
    },
    {
        label: () => h(
            NButton,
            {
                text: true,
                size: "small",
                style: "width: 100%",
                onClick: () => { toggleDark(); showMobileMenu.value = false; }
            },
            {
                default: () => isDark.value ? t('light') : t('dark'),
                icon: () => h(
                    NIcon, { component: isDark.value ? LightModeFilled : DarkModeFilled }
                )
            }
        ),
        key: "theme"
    },
    {
        label: () => h(
            NButton,
            {
                text: true,
                size: "small",
                style: "width: 100%",
                onClick: () => {
                    localeCache.value == 'zh' ? changeLocale('en') : changeLocale('zh');
                    showMobileMenu.value = false;
                }
            },
            {
                default: () => localeCache.value == 'zh' ? "English" : "中文",
                icon: () => h(
                    NIcon, { component: Language }
                )
            }
        ),
        key: "lang"
    },
    {
        label: () => h(
            NButton,
            {
                text: true,
                size: "small",
                style: "width: 100%",
                tag: "a",
                target: "_blank",
                href: "https://github.com/dreamhunter2333/cloudflare_temp_email",
            },
            {
                default: () => "Github",
                icon: () => h(NIcon, { component: GithubAlt })
            }
        ),
        key: "github"
    }
]);

const copy = async () => {
    try {
        await toClipboard(settings.value.address)
        message.success(t('copied'));
    } catch (e) {
        message.error(e.message || "error");
    }
}

const generateNameLoading = ref(false);
const generateName = async () => {
    try {
        generateNameLoading.value = true;
        const { faker } = await import('https://esm.sh/@faker-js/faker');
        emailName.value = faker.person
            .fullName()
            .replace(/\s+/g, '.')
            .replace(/[^a-zA-Z0-9.]/g, '')
            .toLowerCase();
    } catch (error) {
        message.error(error.message || "error");
    } finally {
        generateNameLoading.value = false;
    }
};

const newEmail = async () => {
    try {
        const res = await api.fetch(
            `/api/new_address`
            + `?name=${emailName.value || ''}`
            + `&domain=${emailDomain.value || ''}`
        );
        jwt.value = res["jwt"];
        await api.getSettings();
        showNewEmail.value = false;
        showPassword.value = true;
    } catch (error) {
        message.error(error.message || "error");
    }
};

const deleteAccount = async () => {
    try {
        await api.fetch(`/api/delete_address`, {
            method: 'DELETE'
        });
        jwt.value = '';
        location.reload()
    } catch (error) {
        message.error(error.message || "error");
    }
};

onMounted(async () => {
    await api.getOpenSettings(message);
    emailDomain.value = openSettings.value.domains ? openSettings.value.domains[0].value : "";
    await api.getSettings();
});
</script>

<template>
    <div>
        <n-page-header>
            <template #title>
                <h3>{{ t('title') }}</h3>
            </template>
            <template #avatar>
                <n-avatar style="margin-left: 10px;" src="/logo.png" />
            </template>
            <template #extra>
                <n-space>
                    <n-menu v-if="!isMobile" mode="horizontal" :options="menuOptions" />
                    <n-button v-else :text="true" @click="showMobileMenu = !showMobileMenu" style="margin-right: 10px;">
                        <template #icon>
                            <n-icon :component="MenuFilled" />
                        </template>
                        {{ t('menu') }}
                    </n-button>
                </n-space>
            </template>
        </n-page-header>
        <n-drawer v-model:show="showMobileMenu" placement="top" style="height: 100vh;">
            <n-drawer-content :title="t('menu')" closable>
                <n-menu :options="menuOptions" />
            </n-drawer-content>
        </n-drawer>
        <div v-if="!isAdminRoute">
            <n-card v-if="!settings.fetched">
                <n-skeleton style="height: 50vh" />
            </n-card>
            <div v-else-if="settings.address">
                <n-alert v-if="settings.has_v1_mails" type="warning" show-icon closable>
                    <span>
                        <n-button tag="a" target="_blank" tertiary type="info" size="small"
                            href="https://mail-v1.awsl.uk">
                            <b>{{ t('mailV1Alert') }} </b>
                        </n-button>
                    </span>
                </n-alert>
                <n-alert type="info" show-icon>
                    <span>
                        <b>{{ t('yourAddress') }} <b>{{ settings.address }}</b></b>
                        <n-button style="margin-left: 10px" @click="router.push('/send')" size="small" tertiary round
                            type="primary">
                            <n-icon :component="SendFilled" /> {{ t('sendMail') }}
                        </n-button>
                        <n-button style="margin-left: 10px" @click="copy" size="small" tertiary round type="primary">
                            <n-icon :component="Copy" /> {{ t('copy') }}
                        </n-button>
                    </span>
                </n-alert>
            </div>
            <n-card v-else>
                <n-result status="info" :description="t('pleaseGetNewEmail')">
                    <template #footer>
                        <n-alert v-if="jwt" type="warning" show-icon>
                            <span>{{ t('fetchAddressError') }}</span>
                        </n-alert>
                        <n-button @click="showLogin = true" tertiary round type="primary">
                            {{ t('login') }}
                        </n-button>
                        <n-button @click="showNewEmail = true" tertiary round type="primary">
                            {{ t('getNewEmail') }}
                        </n-button>
                    </template>
                </n-result>
            </n-card>
        </div>
        <n-modal v-model:show="showNewEmail" preset="dialog" title="Dialog">
            <template #header>
                <div>{{ t('getNewEmail') }}</div>
            </template>
            <n-spin :show="generateNameLoading">
                <span>
                    <p>{{ t("getNewEmailTip1") }}</p>
                    <p>{{ t("getNewEmailTip2") }}</p>
                    <p>{{ t("getNewEmailTip3") }}</p>
                </span>
                <n-button @click="generateName" style="margin-bottom: 10px;">
                    {{ t('generateName') }}
                </n-button>
                <n-input-group>
                    <n-input-group-label v-if="openSettings.prefix">
                        {{ openSettings.prefix }}
                    </n-input-group-label>
                    <n-input v-model:value="emailName" />
                    <n-input-group-label>@</n-input-group-label>
                    <n-select v-model:value="emailDomain" :consistent-menu-width="false"
                        :options="openSettings.domains" />
                </n-input-group>
            </n-spin>
            <template #action>
                <n-button @click="showNewEmail = false">
                    {{ t('cancel') }}
                </n-button>
                <n-button @click="newEmail" type="primary" :loading="loading">
                    {{ t('ok') }}
                </n-button>
            </template>
        </n-modal>
        <n-modal v-model:show="showPassword" preset="dialog" title="Dialog">
            <template #header>
                <div>{{ t("password") }}</div>
            </template>
            <span>
                <p>{{ t("passwordTip") }}</p>
            </span>
            <n-card>
                <b>{{ jwt }}</b>
            </n-card>
            <template #action>
            </template>
        </n-modal>
        <n-modal v-model:show="showLogin" preset="dialog" title="Dialog">
            <template #header>
                <div>{{ t('login') }}</div>
            </template>
            <AdminContact />
            <n-input v-model:value="password" type="textarea" :autosize="{
                minRows: 3
            }" />
            <template #action>
                <n-button @click="login" :loading="loading" size="small" tertiary round type="primary">
                    {{ t('login') }}
                </n-button>
            </template>
        </n-modal>
        <n-modal v-model:show="showLogout" preset="dialog" title="Dialog">
            <template #header>
                <div>{{ t('logout') }}</div>
            </template>
            <p>{{ t('logoutConfirm') }}</p>
            <template #action>
                <n-button :loading="loading" @click="logout" size="small" tertiary round type="primary">
                    {{ t('logout') }}
                </n-button>
            </template>
        </n-modal>
        <n-modal v-model:show="showDelteAccount" preset="dialog" title="Dialog">
            <template #header>
                <div>{{ t('delteAccount') }}</div>
            </template>
            <p>{{ t('delteAccountConfirm') }}</p>
            <template #action>
                <n-button :loading="loading" @click="deleteAccount" size="small" tertiary round type="error">
                    {{ t('delteAccount') }}
                </n-button>
            </template>
        </n-modal>
        <n-modal v-model:show="showAuth" :closable="false" :closeOnEsc="false" :maskClosable="false" preset="dialog"
            title="Dialog">
            <template #header>
                <div>{{ t('accessHeader') }}</div>
            </template>
            <p>{{ t('accessTip') }}</p>
            <n-input v-model:value="auth" type="textarea" :autosize="{
                minRows: 3
            }" />
            <template #action>
                <n-button :loading="loading" @click="authFunc" type="primary">
                    {{ t('ok') }}
                </n-button>
            </template>
        </n-modal>
    </div>
</template>

<style scoped>
.n-layout-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
}

.mobile-menu-button {
    width: 100%;
    text-align: left;
}

.n-alert {
    margin-top: 10px;
    margin-bottom: 10px;
    text-align: center;
}

.n-card {
    margin-top: 10px;
}
</style>
