<ul class="navbar-nav ml-auto">
  {#if $conversation.$isUserJoined()}
    <li class="nav-item dropdown" bind:this={notificationsDropdown}>
      <a
        class="nav-link dropdown-toggle"
        href="javascript:void(0)"
        id="notificationsDropdown"
        title="Notifications"
        role="button"
        data-toggle="dropdown"
        aria-haspopup="true"
        aria-expanded="false"
      >
        {#if $conversation.$notifications === Conversation.NOTIFICATIONS_ALL}
          <span>
            <i class="fas fa-bell" />
          </span>
        {:else if $conversation.$notifications === Conversation.NOTIFICATIONS_MENTIONS}
          <span>
            <i class="far fa-bell" />
          </span>
        {:else if $conversation.$notifications === Conversation.NOTIFICATIONS_DIRECT}
          <span>
            <i class="far fa-bell" />
          </span>
        {:else if $conversation.$notifications === Conversation.NOTIFICATIONS_NONE}
          <span>
            <i class="fas fa-bell-slash" />
          </span>
        {/if}
        <span class="sr-only">
          ({Conversation.NOTIFICATIONS_NAME[$conversation.$notifications]})
        </span>
      </a>
      <div
        class="dropdown-menu dropdown-menu-right"
        aria-labelledby="notificationsDropdown"
      >
        <h6 class="dropdown-header">Notification Setting</h6>
        {#each Object.keys(Conversation.NOTIFICATIONS_NAME).map(parseFloat) as key}
          <a
            class="dropdown-item {$conversation.$notifications === key ? 'active' : ''}"
            href="javascript:void(0)"
            on:click={() => setNotifications(key)}
          >
            {Conversation.NOTIFICATIONS_NAME[key]}
          </a>
        {/each}
      </div>
    </li>
  {/if}
  <li class="nav-item {$view === 'conversation' ? 'active' : ''}">
    <a class="nav-link" href="/c/{$conversation.guid}" title="Conversation">
      <i class="fas fa-comments" />
      {#if $view === 'conversation'}
        <span class="sr-only">(current)</span>
      {/if}
    </a>
  </li>
  <li class="nav-item {$view === 'people' ? 'active' : ''}">
    <a
      class="nav-link"
      href="/c/{$conversation.guid}/people"
      title="People ({$conversation.acFull.length})"
    >
      <span class="fa-layers fa-fw">
        <i class="fas fa-users" />
        {#if $conversation.mode === Conversation.MODE_CHAT}
          <span
            class="fa-layers-counter fa-layers-bottom-right bg-info"
            style="transform: scale(0.6); bottom: -.4em; right: -.4em;"
          >
            {$conversation.acFull.length}
          </span>
        {/if}
      </span>
      {#if $view === 'people'}
        <span class="sr-only">(current)</span>
      {/if}
    </a>
  </li>
  {#if $conversation.$isUserJoined()}
    <li class="nav-item {$view === 'settings' ? 'active' : ''}">
      <a
        class="nav-link"
        href="/c/{$conversation.guid}/settings"
        title="Settings"
      >
        <i class="fas fa-cog" />
        {#if $view === 'settings'}
          <span class="sr-only">(current)</span>
        {/if}
      </a>
    </li>
  {/if}
</ul>

<script>
  import { navigate } from '../Services/router';
  import Conversation from '../Entities/Tunnelgram/Conversation';
  import { Dropdown } from '../Services/Val/BSN';
  import { conversation, view } from '../stores';

  let notificationsDropdown;
  let notificationsDropdownComponent;
  $: if (notificationsDropdown && !notificationsDropdownComponent) {
    notificationsDropdownComponent = new Dropdown(notificationsDropdown);
  } else if (!notificationsDropdown && notificationsDropdownComponent) {
    notificationsDropdownComponent = null;
  }

  async function setNotifications(key) {
    const notif = $conversation.$saveNotificationSetting(key);
    $conversation = $conversation;
    await notif;
    $conversation = $conversation;
  }
</script>
