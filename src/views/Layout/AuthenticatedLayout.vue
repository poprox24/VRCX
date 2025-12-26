<template>
    <template v-if="watchState.isLoggedIn">
        <el-splitter class="nav-inner-splitter" @resize-end="handleResizeEnd">
            <el-splitter-panel
                class="nav-layout-panel"
                :size="navPanelSize"
                :min="navPanelMin"
                :max="navPanelMax"
                :resizable="!isNavCollapsed">
                <NavMenu></NavMenu>
            </el-splitter-panel>
            <el-splitter-panel>
                <RouterView></RouterView>
            </el-splitter-panel>

            <el-splitter-panel v-if="isSideBarTabShow" :min="250" :max="700" :size="asideWidth" collapsible>
                <Sidebar></Sidebar>
            </el-splitter-panel>
        </el-splitter>

        <!-- ## Dialogs ## -->
        <UserDialog></UserDialog>

        <WorldDialog></WorldDialog>

        <AvatarDialog></AvatarDialog>

        <GroupDialog></GroupDialog>

        <GroupMemberModerationDialog></GroupMemberModerationDialog>

        <InviteGroupDialog></InviteGroupDialog>

        <FullscreenImagePreview></FullscreenImagePreview>

        <PreviousInstancesInfoDialog></PreviousInstancesInfoDialog>

        <LaunchDialog></LaunchDialog>

        <LaunchOptionsDialog></LaunchOptionsDialog>

        <FriendImportDialog></FriendImportDialog>

        <WorldImportDialog></WorldImportDialog>

        <AvatarImportDialog></AvatarImportDialog>

        <ChooseFavoriteGroupDialog></ChooseFavoriteGroupDialog>

        <VRChatConfigDialog></VRChatConfigDialog>

        <PrimaryPasswordDialog></PrimaryPasswordDialog>

        <SendBoopDialog></SendBoopDialog>

        <ChangelogDialog></ChangelogDialog>
    </template>
</template>

<script setup>
    import { computed, onMounted, ref } from 'vue';
    import { storeToRefs } from 'pinia';
    import { useRouter } from 'vue-router';
    import { watch } from 'vue';

    import { useAppearanceSettingsStore } from '../../stores';
    import { watchState } from '../../service/watchState';

    import AvatarDialog from '../../components/dialogs/AvatarDialog/AvatarDialog.vue';
    import AvatarImportDialog from '../Favorites/dialogs/AvatarImportDialog.vue';
    import ChangelogDialog from '../Settings/dialogs/ChangelogDialog.vue';
    import ChooseFavoriteGroupDialog from '../../components/dialogs/ChooseFavoriteGroupDialog.vue';
    import FriendImportDialog from '../Favorites/dialogs/FriendImportDialog.vue';
    import FullscreenImagePreview from '../../components/FullscreenImagePreview.vue';
    import GroupDialog from '../../components/dialogs/GroupDialog/GroupDialog.vue';
    import GroupMemberModerationDialog from '../../components/dialogs/GroupDialog/GroupMemberModerationDialog.vue';
    import InviteGroupDialog from '../../components/dialogs/InviteGroupDialog.vue';
    import LaunchDialog from '../../components/dialogs/LaunchDialog.vue';
    import LaunchOptionsDialog from '../Settings/dialogs/LaunchOptionsDialog.vue';
    import NavMenu from '../../components/NavMenu.vue';
    import PreviousInstancesInfoDialog from '../../components/dialogs/PreviousInstancesDialog/PreviousInstancesInfoDialog.vue';
    import PrimaryPasswordDialog from '../Settings/dialogs/PrimaryPasswordDialog.vue';
    import SendBoopDialog from '../../components/dialogs/SendBoopDialog.vue';
    import Sidebar from '../Sidebar/Sidebar.vue';
    import UserDialog from '../../components/dialogs/UserDialog/UserDialog.vue';
    import VRChatConfigDialog from '../Settings/dialogs/VRChatConfigDialog.vue';
    import WorldDialog from '../../components/dialogs/WorldDialog/WorldDialog.vue';
    import WorldImportDialog from '../Favorites/dialogs/WorldImportDialog.vue';
    import configRepository from '../../service/config';

    const router = useRouter();

    const appearanceStore = useAppearanceSettingsStore();
    const { setAsideWidth } = appearanceStore;
    const { asideWidth, isSideBarTabShow, isNavCollapsed } = storeToRefs(appearanceStore);

    const NAV_COLLAPSED_WIDTH = 64;
    const NAV_MIN_WIDTH = 200;
    const NAV_MAX_WIDTH = 360;
    const NAV_DEFAULT_WIDTH = 240;
    const NAV_WIDTH_KEY = 'VRCX_navWidth';

    const navWidth = ref(NAV_DEFAULT_WIDTH);
    const isNavResizing = ref(false);

    const clampNavWidth = (width) => Math.min(Math.max(width, NAV_MIN_WIDTH), NAV_MAX_WIDTH);

    const navPanelSize = computed(() => (isNavCollapsed.value ? NAV_COLLAPSED_WIDTH : navWidth.value));
    const navPanelMin = computed(() => (isNavCollapsed.value ? NAV_COLLAPSED_WIDTH : NAV_MIN_WIDTH));
    const navPanelMax = computed(() => (isNavCollapsed.value ? NAV_COLLAPSED_WIDTH : NAV_MAX_WIDTH));

    const persistNavWidth = () => {
        if (!isNavCollapsed.value) {
            configRepository.setInt(NAV_WIDTH_KEY, navWidth.value);
        }
    };

    const handleResizeEnd = (index, sizes) => {
        if (Array.isArray(sizes)) {
            if (index === 0 && !isNavCollapsed.value) {
                navWidth.value = clampNavWidth(sizes[0]);
                persistNavWidth();
            }
            if (index === 1) {
                setAsideWidth(sizes);
            }
        }
        isNavResizing.value = false;
    };

    onMounted(async () => {
        const savedWidth = await configRepository.getInt(NAV_WIDTH_KEY, NAV_DEFAULT_WIDTH);
        navWidth.value = clampNavWidth(savedWidth || NAV_DEFAULT_WIDTH);
    });

    watch(
        () => watchState.isLoggedIn,
        (isLoggedIn) => {
            if (!isLoggedIn) {
                router.replace({ name: 'login' });
            }
        },
        { immediate: true }
    );
</script>
