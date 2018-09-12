今天写了这样一段代码，希望路由可以这样链式 redirect ：

'' --> 'frontend' --> 'frontend/overview' --> 'frontend/overview/general'

```
  {
    path: '',
    redirectTo: 'frontend',
    pathMatch: 'full',
  },
  {
    path: 'frontend',
    redirectTo: 'frontend/overview',
    pathMatch: 'full',
  },
  {
    path: 'frontend/overview',
    redirectTo: 'frontend/overview/general',
    pathMatch: 'full',
  },
```

但是却不能成功跳转。为什么勒？

网上基本上找不到 Angular Router redirect 相关的资料，只能自己动手打开 chrome devTool。

下面是 Angular Router 匹配当前路由和注册路由配置的方法，它的最后一个字段是 allowRedirects ，表示时候可以接受跳转。

```
  ApplyRedirects.prototype.expandSegment = function (ngModule, segmentGroup, routes, segments, outlet, allowRedirects) {
      var _this = this;
      var /** @type {?} */ routes$ = of.apply(void 0, routes);
      var /** @type {?} */ processedRoutes$ = map.call(routes$, function (r) {
          var /** @type {?} */ expanded$ = _this.expandSegmentAgainstRoute(ngModule, segmentGroup, routes, r, segments, outlet, allowRedirects);
          return _catch.call(expanded$, function (e) {
              if (e instanceof NoMatch) {
                  return of(null);
              }
              throw e;
          });
      });
      var /** @type {?} */ concattedProcessedRoutes$ = concatAll.call(processedRoutes$);
      var /** @type {?} */ first$ = first.call(concattedProcessedRoutes$, function (s) { return !!s; });
      return _catch.call(first$, function (e, _) {
          if (e instanceof EmptyError) {
              if (_this.noLeftoversInUrl(segmentGroup, segments, outlet)) {
                  return of(new UrlSegmentGroup([], {}));
              }
              throw new NoMatch(segmentGroup);
          }
          throw e;
      });
  };
```

第一次跳转之前，会匹配当前路由，也就是 '' 空路由。

```
  ApplyRedirects.prototype.expandSegmentGroup = function (ngModule, routes, segmentGroup, outlet) {
      if (segmentGroup.segments.length === 0 && segmentGroup.hasChildren()) {
          return map.call(this.expandChildren(ngModule, routes, segmentGroup), function (children) { return new UrlSegmentGroup([], children); });
      }
      return this.expandSegment(ngModule, segmentGroup, routes, segmentGroup.segments, outlet, true);
  };
```

这时候 allowRedirects 的值为 true 。

等到匹配到空路由，根据路由配置会 redirect 为 'front' 。接下来会继续从路由配置中寻找 'front' 。

但是在经历过一次redirect之后，路由匹配的方法变成了另一个。

```
  ApplyRedirects.prototype.expandRegularSegmentAgainstRouteUsingRedirect = function (ngModule, segmentGroup, routes, route, segments, outlet) {
      var _this = this;
      var _a = match(segmentGroup, route, segments), matched = _a.matched, consumedSegments = _a.consumedSegments, lastChild = _a.lastChild, positionalParamSegments = _a.positionalParamSegments;
      if (!matched)
          return noMatch(segmentGroup);
      var /** @type {?} */ newTree = this.applyRedirectCommands(consumedSegments, /** @type {?} */ ((route.redirectTo)), /** @type {?} */ (positionalParamSegments));
      if (((route.redirectTo)).startsWith('/')) {
          return absoluteRedirect(newTree);
      }
      return mergeMap.call(this.lineralizeSegments(route, newTree), function (newSegments) {
          return _this.expandSegment(ngModule, segmentGroup, routes, newSegments.concat(segments.slice(lastChild)), outlet, false);
      });
  };
```

请看最后一行， allowRedirects 参数变成了false。

我们再看看这个 allowRedirects 参数到底有什么用。

```
  ApplyRedirects.prototype.expandSegmentAgainstRoute = function (ngModule, segmentGroup, routes, route, paths, outlet, allowRedirects) {
      if (getOutlet(route) !== outlet) {
          return noMatch(segmentGroup);
      }
      if (route.redirectTo === undefined) {
          return this.matchSegmentAgainstRoute(ngModule, segmentGroup, route, paths);
      }
      if (allowRedirects && this.allowRedirects) {
          return this.expandSegmentAgainstRouteUsingRedirect(ngModule, segmentGroup, routes, route, paths, outlet);
      }
      return noMatch(segmentGroup);
  };
```

如果 allowRedirects 为 false ，那么就会返回 noMatch 哦！

所以在 Angular Router 中，链式 redirect 是行不通的！


