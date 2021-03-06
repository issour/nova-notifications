<template>
    <dropdown class="ml-auto h-9 flex items-center dropdown-right">
        <dropdown-trigger class="h-9 flex items-center">
        <span class="relative">
            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" class="ml-2 text-90 bg-gray-800 w-8">
                <path
                    d="M15 19a3 3 0 0 1-6 0H4a1 1 0 0 1 0-2h1v-6a7 7 0 0 1 4.02-6.34 3 3 0 0 1 5.96 0A7 7 0 0 1 19 11v6h1a1 1 0 0 1 0 2h-5zm-4 0a1 1 0 0 0 2 0h-2zm0-12.9A5 5 0 0 0 7 11v6h10v-6a5 5 0 0 0-4-4.9V5a1 1 0 0 0-2 0v1.1z"/>
            </svg>
            <span
                class="absolute -mt-4 -mr-1 text-xs bg-danger text-danger-light text-sm font-bold px-1 shadow-lg rounded-lg"
                style="right: 0;"
                v-if="count > 0">
                <span v-if="count < 9">{{ count }}</span>
                <span v-else>9+</span>
            </span>
        </span>
        </dropdown-trigger>

        <dropdown-menu slot="menu" width="600" direction="rtl">
            <loading-view :loading="isLoading">
                <div class="flex justify-between bg-40 text-90 p-4">
                    <h3>Notifications</h3>

                    <button v-if="count !== 0" class="btn" @click="markAllAsRead()">
                        mark all as read
                    </button>
                </div>
                <p v-if="count === 0" class="block p-3">
                    No new notifications.
                </p>
                <scroll-wrap v-else height="350">
                    <slot>
                        <notification-item
                            :ref="'notification-' + notification.id"
                            v-for="notification in notifications"
                            :key="notification.id"
                            :notification="notification"
                        >
                        </notification-item>
                    </slot>
                </scroll-wrap>
            </loading-view>
        </dropdown-menu>
    </dropdown>
</template>

<script>
    export default {
        data() {
            return {
                count: 0,
                notifications: [],
                isLoading: true,
            }
        },
        mounted() {
            const that = this

            this.loadNotifications(function (response) {
                console.log(Nova.config.user_model_namespace)
                Echo.private(Nova.config.user_model_namespace + '.' + response.data.user_id)
                    .notification(that.notificationReceived)
            })

            Nova.$on('notification-read', function (e) {
                that.notifications[e.notification.id] = e.notification
                that.count--;
            })
        },
        methods: {
            loadNotifications: function (callback) {
                axios
                    .get('/nova-vendor/nova-notifications/unread')
                    .then(response => {
                        this.isLoading = false
                        this.count = response.data.count
                        this.notifications = response.data.notifications

                        if (callback) {
                            callback(response)
                        }
                    })
            },
            notificationReceived: function (notification) {
                this.loadNotifications()

                const that = this

                let level = 'info'
                const levels = ['success', 'info', 'error']

                if (levels.indexOf(notification.level) !== -1) {
                    level = notification.level
                }

                this.$toasted.show(notification.title, {
                    type: level,
                    keepOnHover: true,
                    action: [{
                        text: 'Mark as Read',
                        onClick: (e, toast) => {
                            that.$refs['notification-' + notification.id][0].markAsRead()
                            toast.goAway(0);
                        }
                    }, {
                        text: 'Cancel',
                        onClick: (e, toast) => {
                            toast.goAway(0);
                        }
                    }],
                })
            },
            markAllAsRead: function () {
                axios
                    .patch('/nova-vendor/nova-notifications')
                    .then(response => {
                        this.notifications = []
                        this.count = 0
                    })
            }
        }
    }
</script>

<style scoped>
    #max-height {
        max-height: 30rem;
    }

    .bg-light-gray {
        background-color: #EEEEEE;
    }

    .bg-light-gray:hover {
        background-color: #fefefe;
    }
</style>
