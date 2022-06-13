<script>
  import LogoTitle from './LogoTitle.svelte';
  import BadgeCounter from './BadgeCounter.svelte';
  import Authorization from '../Authorization.svelte';
  import AuthorizationSimpleProfileMenu from '../AuthorizationSimpleProfileMenu.svelte';
  import TopNavDropDown from '../TopNavDropDown.svelte';
  import ContextSwitcher from './ContextSwitcher.svelte';
  import ProductSwitcher from './ProductSwitcher.svelte';
  import GlobalSearch from './GlobalSearch.svelte';
  import GlobalSearchCentered from './GlobalSearchCentered.svelte';
  import TopNavNode from './TopNavNode.svelte';
  import isEqual from 'lodash';

  import {
    beforeUpdate,
    createEventDispatcher,
    onMount,
    getContext,
  } from 'svelte';
  import { LuigiAuth, LuigiConfig, LuigiI18N } from '../core-api';
  import {
    AuthHelpers,
    NavigationHelpers,
    RoutingHelpers,
    StateHelpers,
    EventListenerHelpers,
    GenericHelpers,
  } from '../utilities/helpers';
  import { SemiCollapsibleNavigation } from './services/semi-collapsed-navigation';

  const dispatch = createEventDispatcher();
  export let authorizationEnabled;
  export let autologinEnabled;
  export let isLoggedIn = false;
  export let hideNavComponent;
  export let responsiveNavSetting;
  export let profileTypeSettings;
  export let showGlobalNav;
  export let pathData;
  let previousPathData;
  export let pathParams;
  export let dropDownStates = {};
  export let children;
  export let selectedNode;
  export let visibleNodeCount;
  export let globalNavNodeCount;
  export let totalBadgeNode;
  export let isProductSwitcherAvailable;
  export let productSwitcherConfig;
  export let openMobileDropDown;
  export let nodeForMobile;
  export let profileItemsAvailable;
  export let userInfo = {};
  export let urlAuthError;
  export let globalSearchConfig;
  export let isGlobalSearchCentered;
  export let isSearchFieldVisible;
  export let inputElem;
  export let luigiCustomSearchRenderer__slot;
  export let displaySearchResult;
  export let displayCustomSearchResult;
  export let searchResult;
  export let burgerTooltip;

  let logoUrl;
  let title;

  let store = getContext('store');
  let contextSwitcherToggle = false;
  let selectedLabel;
  let defaultLabelContextSwitcher;
  let contextSwitcherConfig = LuigiConfig.getConfigValue(
    'navigation.contextSwitcher'
  );
  export let addNavHrefForAnchor = LuigiConfig.getConfigBooleanValue(
    'navigation.addNavHrefs'
  );

  const api = {};

  const updateWrapper = () => {
    const wrapper = document.getElementById('lui-shellbar-wrapper');

    wrapper.api = api;
    if(wrapper.update) {
      const state = {
        app: {
          authorizationEnabled,
          profileItemsAvailable,
          autologinEnabled,
          isProductSwitcherAvailable,
          hideNavComponent,
          responsiveNavSetting,
          profileTypeSettings,
          productSwitcherConfig,
          globalSearchConfig,
          isGlobalSearchCentered,
          showGlobalNav,
          addNavHrefForAnchor,
          contextSwitcherConfig,
          logoUrl,
          title,
        },
        navigation: {
          children,
          selectedNode,
          visibleNodeCount,
          totalBadgeNode,
          globalNavNodeCount,
          previousPathData
        }
      };
        wrapper.update(state);

    }
  };

  const setTopNavData = async () => {
    if (pathData && 0 < pathData.length) {
      const tnd = await NavigationHelpers.generateTopNavNodes(pathData);
      children = tnd.children;
      selectedNode = tnd.selectedNode;
      visibleNodeCount = tnd.visibleNodeCount;
      totalBadgeNode = tnd.totalBadgeNode;
      globalNavNodeCount = tnd.globalNavNodeCount;
      window.TOPNAVDATA = tnd.children;
      previousPathData = pathData;

      updateWrapper();
    }
  };

  const setLoggedInState = () => {
    isLoggedIn = AuthHelpers.isLoggedIn();
  };

  onMount(() => {
    const wrapper = document.getElementById('lui-shellbar-wrapper');
    if(!document.querySelector('#app #lui-shellbar-wrapper')) {
      document.querySelector('#app').appendChild(wrapper);
    }
    StateHelpers.doOnStoreChange(
      store,
      () => {
        authorizationEnabled = LuigiAuth.isAuthorizationEnabled();
        profileItemsAvailable =
          LuigiConfig.getConfigValue('navigation.profile');
        autologinEnabled = !Boolean(
          LuigiConfig.getConfigValue('auth.disableAutoLogin')
        );
        isProductSwitcherAvailable = LuigiConfig.getConfigValue(
          'navigation.productSwitcher'
        );
        hideNavComponent = LuigiConfig.getConfigBooleanValue(
          'settings.hideNavigation'
        );
        responsiveNavSetting = LuigiConfig.getConfigValue(
          'settings.responsiveNavigation'
        );
        profileTypeSettings = LuigiConfig.getConfigValue(
          'settings.profileType'
        );
        productSwitcherConfig = NavigationHelpers.getProductSwitcherConfig();
        globalSearchConfig = LuigiConfig.getConfigValue('globalSearch');
        isGlobalSearchCentered =
          globalSearchConfig &&
          globalSearchConfig.searchFieldCentered &&
          GenericHelpers.requestExperimentalFeature(
            'globalSearchCentered',
            true
          );
        showGlobalNav =
          LuigiConfig.getConfigBooleanValue('settings.globalSideNavigation') &&
          GenericHelpers.requestExperimentalFeature('globalNav', true);
        addNavHrefForAnchor = LuigiConfig.getConfigBooleanValue(
          'navigation.addNavHrefs'
        );
        contextSwitcherConfig = LuigiConfig.getConfigValue(
          'navigation.contextSwitcher'
        );

        logoUrl = LuigiConfig.getConfigValue('settings.header.logo') || 'data:image/svg+xml;base64,PHN2ZyBkYXRhLW5hbWU9IkxheWVyIDEiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyIgdmlld0JveD0iMCAwIDQxMi4zOCAyMDQiPjxkZWZzPjxsaW5lYXJHcmFkaWVudCBpZD0iYSIgeDE9IjIwNi4xOSIgeDI9IjIwNi4xOSIgeTI9IjIwNCIgZ3JhZGllbnRVbml0cz0idXNlclNwYWNlT25Vc2UiPjxzdG9wIG9mZnNldD0iMCIgc3RvcC1jb2xvcj0iIzAwYjhmMSIvPjxzdG9wIG9mZnNldD0iLjAyIiBzdG9wLWNvbG9yPSIjMDFiNmYwIi8+PHN0b3Agb2Zmc2V0PSIuMzEiIHN0b3AtY29sb3I9IiMwZDkwZDkiLz48c3RvcCBvZmZzZXQ9Ii41OCIgc3RvcC1jb2xvcj0iIzE3NzVjOCIvPjxzdG9wIG9mZnNldD0iLjgyIiBzdG9wLWNvbG9yPSIjMWM2NWJmIi8+PHN0b3Agb2Zmc2V0PSIxIiBzdG9wLWNvbG9yPSIjMWU1ZmJiIi8+PC9saW5lYXJHcmFkaWVudD48L2RlZnM+PHBhdGggZD0iTTAgMjA0aDIwOC40MUw0MTIuMzggMEgwdjIwNCIgZmlsbD0idXJsKCNhKSIgZmlsbC1ydWxlPSJldmVub2RkIi8+PHBhdGggZD0iTTI0NC43MyAzOC4zNmgtNDAuNnY5Ni41MmwtMzUuNDYtOTYuNTVoLTM1LjE2bC0zMC4yNyA4MC43MkMxMDAgOTguNyA3OSA5MS42NyA2Mi40IDg2LjQgNTEuNDYgODIuODkgMzkuODUgNzcuNzIgNDAgNzJjLjA5LTQuNjggNi4yMy05IDE4LjM4LTguMzggOC4xNy40MyAxNS4zNyAxLjA5IDI5LjcxIDhsMTQuMS0yNC41NUM4OS4wNiA0MC40MiA3MSAzNi4yMSA1Ni4xNyAzNi4xOWgtLjA5Yy0xNy4yOCAwLTMxLjY4IDUuNi00MC42IDE0LjgzQTM0LjIzIDM0LjIzIDAgMDA1Ljc3IDc0LjdDNS41NCA4Ny4xNSAxMC4xMSA5NiAxOS43MSAxMDNjOC4xIDUuOTQgMTguNDYgOS43OSAyNy42IDEyLjYyIDExLjI3IDMuNDkgMjAuNDcgNi41MyAyMC4zNiAxM0E5LjU3IDkuNTcgMCAwMTY1IDEzNWMtMi44MSAyLjktNy4xMyA0LTEzLjA5IDQuMS0xMS40OS4yNC0yMC0xLjU2LTMzLjYxLTkuNTlMNS43NyAxNTQuNDJhOTMuNzcgOTMuNzcgMCAwMDQ2IDEyLjIyaDIuMTFjMTQuMjQtLjI1IDI1Ljc0LTQuMzEgMzQuOTItMTEuNzEuNTMtLjQxIDEtLjg0IDEuNDktMS4yOGwtNC4xMiAxMC44NUgxMjNsNi4xOS0xOC44MmE2Ny40NiA2Ny40NiAwIDAwMjEuNjggMy40MyA2OC4zMyA2OC4zMyAwIDAwMjEuMTYtMy4yNWw2IDE4LjY0aDYwLjE0di0zOWgxMy4xMWMzMS43MSAwIDUwLjQ2LTE2LjE1IDUwLjQ2LTQzLjIgMC0zMC4xMS0xOC4yMi00My45NC01Ny4wMS00My45NHpNMTUwLjkxIDEyMWEzNi45MyAzNi45MyAwIDAxLTEzLTIuMjhsMTIuODctNDAuNTloLjIybDEyLjY1IDQwLjcxYTM4LjUgMzguNSAwIDAxLTEyLjc0IDIuMTZ6bTk2LjItMjMuMzNoLTguOTRWNjQuOTFoOC45NGMxMS45MyAwIDIxLjQ0IDQgMjEuNDQgMTYuMTQgMCAxMi42LTkuNTEgMTYuNTctMjEuNDQgMTYuNTciIGZpbGw9IiNmZmYiIGZpbGwtcnVsZT0iZXZlbm9kZCIvPjwvc3ZnPg==';
        title = LuigiConfig.getConfigValue('settings.header.title');

        updateWrapper();
      },
      ['navigation']
    );

    EventListenerHelpers.addEventListener('message', (e) => {
      if ('luigi.navigation.update-badge-counters' === e.data.msg) {
        setTopNavData();
      }
    });
  });

  beforeUpdate(() => {
    if (!previousPathData || previousPathData != pathData) {
      setTopNavData();
    }
    setLoggedInState();
  });

  export let showTopNav;
  $: showTopNav =
    (authorizationEnabled && (!autologinEnabled || isLoggedIn)) ||
    !authorizationEnabled;

  function getSapIconStr(iconString) {
    return NavigationHelpers.renderIconClassName(iconString);
  }

  function hasOpenUIicon(node) {
    return NavigationHelpers.isOpenUIiconName(node.icon);
  }

  function getNodeLabel(node) {
    return LuigiI18N.getTranslation(node.label);
  }

  api.getNodeLabel = (node) => {
    return getNodeLabel(node);
  }

  function getTestId(node) {
    return node.testId
      ? node.testId
      : NavigationHelpers.prepareForTests(node.pathSegment, node.label);
  }

  function getRouteLink(node) {
    return RoutingHelpers.getNodeHref(node, pathParams);
  }

  function resolveTooltipText(node, translation) {
    return NavigationHelpers.generateTooltipText(node, translation);
  }

  export function openMobileProductSwitcher() {
    toggleDropdownState('productSwitcherPopover');
  }

  export function openMobileContextSwitcher() {
    contextSwitcherToggle = !contextSwitcherToggle;
    toggleDropdownState('contextSwitcherPopover');
  }

  export function openMobileTopNavDropDown(node) {
    openMobileDropDown = true;
    nodeForMobile = node;
  }

  export function closeMobileTopNavDropDown() {
    openMobileDropDown = false;
  }

  export function handleClick(node) {
    dispatch('handleClick', { node });
  }

  api.handleClick = (node) => {
    handleClick(node);
  }

  export function handleSearchNavigation(node) {
    dispatch('handleSearchNavigation', { node });
  }

  export function handleClickExternal(event) {
    handleClick(event.detail.node);
  }

  export function toggleDropdownState(name) {
    const ddStates = dropDownStates || {};
    const dropDownState = !ddStates[name];

    closeAllDropdowns();
    closeMobileTopNavDropDown();
    ddStates[name] = dropDownState;

    dropDownStates = ddStates;
  }

  export function toggleDropdownStateExternal(event) {
    toggleDropdownState(event.detail.name);
  }

  function setFocusOnGlobalSearchFieldMobile() {
    if (inputElem) {
      inputElem.focus();
    }
  }

  export function toggleSearch() {
    if (!isSearchFieldVisible)
      setTimeout(() => {
        setFocusOnGlobalSearchFieldMobile();
      });
    dispatch('toggleSearch', {
      isSearchFieldVisible,
      inputElem,
      luigiCustomSearchRenderer__slot,
    });
  }

  export function closeAllDropdowns() {
    const ddStates = dropDownStates || {};
    const keys = Object.keys(ddStates);
    if (keys && keys.length > 0) {
      keys.forEach((k) => {
        ddStates[k] = false;
        dropDownStates = ddStates;
      });
    }
  }

  function burgerClickHandler() {
    if (
      responsiveNavSetting === 'simple' ||
      responsiveNavSetting === 'simpleMobileOnly'
    ) {
      simpleNav();
    } else {
      semicollapsedNav();
    }
    setBurgerTooltip();
  }

  api.burgerClickHandler = () => {
    burgerClickHandler();
  }

  function setBurgerTooltip() {
    if (!NavigationHelpers.getBurgerTooltipConfig()) {
      return;
    }
    const [collapseNavTooltip, expandNavTooltip] =
      NavigationHelpers.getBurgerTooltipConfig();
    if (collapseNavTooltip && expandNavTooltip) {
      if (document.body.classList.contains('lui-simpleSlideInNav')) {
        burgerTooltip = document.body.classList.contains('lui-leftNavToggle')
          ? collapseNavTooltip
          : expandNavTooltip;
      }
      if (document.body.classList.contains('lui-semiCollapsible')) {
        burgerTooltip = document.body.classList.contains('semiCollapsed')
          ? collapseNavTooltip
          : expandNavTooltip;
      }
    }
  }

  export function simpleNav() {
    document.body.classList.toggle('lui-leftNavToggle');
    if (document.getElementsByClassName('fd-tabs').length > 0) {
      dispatch('resizeTabNav', {});
    }
  }

  export function semicollapsedNav() {
    SemiCollapsibleNavigation.buttonClicked();
    if (document.getElementsByClassName('fd-tabs').length > 0) {
      dispatch('resizeTabNav', {});
    }
  }

  export function userInfoUpdate(event) {
    const uInfo = event.detail;
    userInfo = uInfo ? uInfo : {};
  }
</script>

<svelte:window on:click={closeAllDropdowns} on:blur={closeAllDropdowns} />
{#if showTopNav}

{:else}
  <AuthorizationSimpleProfileMenu
    on:toggleDropdownState={() => toggleDropdownState('profilePopover')}
    isHidden={true}
    {addNavHrefForAnchor}
  />
{/if}

<style type="text/scss">
  @import 'styles/variables';

  .hideNavComponent {
    display: none;
  }

  .nav-icon {
    height: 100%;
  }

  .fd-top-nav__icon {
    img {
      max-height: 16px;
      vertical-align: top;
      max-width: 16px;
    }
  }

  img {
    &.fd-top-nav__icon {
      height: 16px;
      min-width: auto;
    }
  }

  .fd-popover {
    .nav-icon {
      height: 2em;
    }
  }

  .lui-burger {
    cursor: pointer;
    color: var(--sapShell_TextColor, #fff);
  }

  @media (min-width: $desktopMinWidth) {
    :global(.lui-mobileOnly) .lui-burger {
      display: none;
    }
  }

  :global(.no-side-nav) {
    :global(.lui-burger) {
      display: none;
    }
  }

  :global(#lui-shellbar-wrapper) {
    display: none;
  }

  :global(#app) {
    :global(#lui-shellbar-wrapper) {
      display: block;
      z-index: 10;
      position: relative;
    }
  }
</style>
