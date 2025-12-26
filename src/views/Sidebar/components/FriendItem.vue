<template>
    <div class="x-friend-item" @click="$emit('click')">
        <template v-if="friend.ref">
            <div
                class="avatar"
                :class="isFriendActiveOrOffline ? undefined : userStatusClass(friend.ref, friend.pendingOffline)">
                <img :src="userImage(friend.ref, true)" loading="lazy" />
            </div>
            <div class="detail h-9">
                <span v-if="!hideNicknames && friend.$nickName" class="name" :style="{ color: friend.ref.$userColour }">
                    {{ friend.ref.displayName }} ({{ friend.$nickName }})
                </span>
                <span v-else class="name" :style="{ color: friend.ref.$userColour }"
                    >{{ friend.ref.displayName }}{{ isGroupByInstance && friend.isVIP ? ' ⭐' : '' }}</span
                >

                <span v-if="isFriendActiveOrOffline" class="extra">{{ friend.ref.statusDescription }}</span>
                <template v-else>
                    <div v-if="friend.pendingOffline" class="extra">
                        <el-icon><WarningFilled /></el-icon> {{ t('side_panel.pending_offline') }}
                    </div>
                    <template v-else-if="isGroupByInstance">
                        <el-icon v-if="isFriendTraveling" class="is-loading" style="margin-right: 3px"
                            ><Loading
                        /></el-icon>
                        <Timer
                            class="extra"
                            :epoch="epoch"
                            :style="isFriendTraveling ? { display: 'inline-block', overflow: 'unset' } : undefined" />
                    </template>
                    <Location v-else class="extra" :location="locationProp" :traveling="travelingProp" :link="false" />
                </template>
            </div>
        </template>
        <template v-else-if="!friend.ref && !isRefreshFriendsLoading">
            <span>{{ friend.name || friend.id }}</span>
            <el-button
                text
                :icon="Close"
                size="small"
                style="margin-left: 5px"
                @click.stop="$emit('confirm-delete-friend', friend.id)">
            </el-button>
        </template>

        <el-skeleton v-else animated class="skeleton" :throttle="100">
            <template #template>
                <div>
                    <el-skeleton-item variant="circle" />
                    <div>
                        <el-skeleton-item variant="text" />
                        <el-skeleton-item variant="text" />
                    </div>
                </div>
            </template>
        </el-skeleton>
    </div>
</template>

<script setup>
    import { Close, Loading, WarningFilled } from '@element-plus/icons-vue';
    import { computed } from 'vue';
    import { storeToRefs } from 'pinia';
    import { useI18n } from 'vue-i18n';

    import { useAppearanceSettingsStore, useFriendStore } from '../../../stores';
    import { userImage, userStatusClass } from '../../../shared/utils';

    const props = defineProps({
        friend: { type: Object, required: true },
        isGroupByInstance: Boolean
    });

    defineEmits(['click', 'confirm-delete-friend']);

    const { hideNicknames } = storeToRefs(useAppearanceSettingsStore());
    const { isRefreshFriendsLoading } = storeToRefs(useFriendStore());
    const { t } = useI18n();

    const isFriendTraveling = computed(() => props.friend.ref?.location === 'traveling');
    const isFriendActiveOrOffline = computed(() => props.friend.state === 'active' || props.friend.state === 'offline');
    const epoch = computed(() =>
        isFriendTraveling.value ? props.friend.ref?.$travelingToTime : props.friend.ref?.$location_at
    );

    const locationProp = computed(() => props.friend.ref?.location || '');
    const travelingProp = computed(() => props.friend.ref?.travelingToLocation || '');
</script>

<style>
    .x-friend-item {
        box-sizing: border-box;
        display: flex;
        align-items: center;
        padding: 5px 5px 5px 0;
        font-size: 12px;
        cursor: pointer;
    }

    .x-friend-item > .avatar {
        position: relative;
        display: inline-block;
        flex: none;
        width: 36px;
        height: 36px;
        margin-right: 10px;
        backdrop-filter: brightness(1.02);
    }

    .x-friend-item > img.avatar,
    img.friends-list-avatar {
        width: unset;
        height: 22.5px;
        margin-right: 0;
        margin-left: 5px;
        border-radius: 2px;
    }

    .x-friend-item > .avatar > img {
        width: 100%;
        height: 100%;
        border-radius: 50%;
        object-fit: cover;
    }

    .x-friend-item > .avatar.active > img {
        filter: grayscale(1);
    }

    .x-friend-item:hover > .avatar.offline > img,
    .x-friend-item:hover > .avatar.active > img {
        filter: none;
    }

    .x-friend-item > .avatar.online.mobile > img,
    .x-friend-item > .avatar.joinme.mobile > img,
    .x-friend-item > .avatar.askme.mobile > img,
    .x-friend-item > .avatar.busy.mobile > img {
        mask-image: url(/images/masks/usercutoutmobile.svg);
    }

    .x-friend-item > .avatar.online.mobile::after,
    .x-friend-item > .avatar.joinme.mobile::after,
    .x-friend-item > .avatar.askme.mobile::after,
    .x-friend-item > .avatar.busy.mobile::after {
        position: absolute;
        right: -2px;
        bottom: 0px;
        width: 14px;
        height: 14px;
        content: '';
        border-radius: 0px;
        mask-image: url(/images/masks/phone.svg);
    }

    .x-friend-item > .avatar.active > img,
    .x-friend-item > .avatar.online > img,
    .x-friend-item > .avatar.joinme > img,
    .x-friend-item > .avatar.askme > img,
    .x-friend-item > .avatar.busy > img,
    .x-friend-item > .avatar.offline > img {
        mask-image: url(/images/masks/usercutout.svg);
    }

    .x-friend-item > .avatar.active::after,
    .x-friend-item > .avatar.online::after,
    .x-friend-item > .avatar.joinme::after,
    .x-friend-item > .avatar.askme::after,
    .x-friend-item > .avatar.busy::after,
    .x-friend-item > .avatar.offline::after {
        position: absolute;
        right: 1px;
        bottom: 1px;
        width: 9px;
        height: 9px;
        content: '';
        background: var(--color-zinc-500);
        border-radius: 50%;
    }

    .x-friend-item > .avatar.active::after {
        background: var(--color-amber-300);
    }

    .x-friend-item > .avatar.online::after {
        background: var(--el-color-success);
    }

    .x-friend-item > .avatar.joinme::after {
        background: var(--el-color-primary);
        mask-image: url(/images/masks/joinme.svg);
    }

    .x-friend-item > .avatar.askme::after {
        background: var(--el-color-warning);
        mask-image: url(/images/masks/askme.svg);
    }

    .x-friend-item > .avatar.busy::after {
        background: var(--el-color-danger);
        mask-image: url(/images/masks/busy.svg);
    }

    .x-friend-item > .avatar.offline::after {
        background: var(--color-zinc-500);
    }

    .x-friend-item.offline > .avatar::after {
        display: none;
    }

    .x-friend-item > .detail {
        flex: 1;
        overflow: hidden;
    }

    .x-friend-item > .detail > .name,
    .x-friend-item > .detail > .extra {
        display: block;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }

    .x-friend-item > .detail > .name {
        font-weight: 500;
        color: var(--el-text-color-primary);
        line-height: 16px;
    }

    .x-friend-item > .detail > .extra,
    .extra {
        color: var(--el-text-color-secondary);
        font-size: 11px;
        & > span > span:first-child {
            scale: 0.9;
            margin-right: 2px;
        }
    }

    .x-friend-item > .vrcplus-icon {
        border: 4px solid var(--el-border-color);
        border-radius: 20px;
        width: 200px;
        height: 200px;
        cursor: pointer;
    }

    .x-friend-item > .current-vrcplus-icon {
        border: 4px solid var(--el-color-success);
        cursor: default;
    }

    .x-friend-item > .vrcplus-icon > img {
        width: 100%;
        height: 100%;
        border-radius: 15px;
        object-fit: cover;
    }

    .skeleton {
        height: 40px;
        width: 100%;
        & > div {
            height: 100%;
            display: flex;
            align-items: center;
            justify-content: space-between;
            & > div {
                width: calc(100% - 48px);
                height: 100%;
                display: flex;
                flex-direction: column;
                justify-content: center;
            }
        }
        .el-skeleton__circle {
            height: 40px;
            width: 40px;
        }
        .el-skeleton__text {
            &:first-child {
                height: 14px;
                margin-bottom: 6px;
                width: 50%;
            }
            &:last-child {
                height: 12px;
            }
        }
    }
</style>
