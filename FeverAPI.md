BazQux Reader API
==========

It's a copy of https://feedafever.com/api in case it will be shut down.

<h2>Authentication</h2>

<p>Without further ado, the Fever API endpoint URL looks like:</p>

<pre><code>http://yourdomain.com/fever/?api</code></pre>

<p>All requests must be authenticated with a <code>POST</code>ed <code>api_key</code>. The value of <code>api_key</code> should be the md5 checksum of the Fever accounts email address and password concatenated with a colon. An example of a valid value for <code>api_key</code> using PHP&#8217;s native <code>md5()</code> function:</p>

<pre><code>$email  = 'you@yourdomain.com';
$pass   = 'b3stp4s4wd3v4';
$api_key = md5($email.':'.$pass);
</code></pre>

<p>A user may specify that <code>https</code> be used to connect to their Fever installation for additional security but you should not assume that all Fever installations support <code>https</code>.</p>

<p>The default response is a JSON object containing two members:</p>

<ul>
<li><code>api_version</code> contains the version of the API responding (positive integer)</li>
<li><code>auth</code> whether the request was successfully authenticated (boolean integer)</li>
</ul>

<p>The API can also return XML by passing <code>xml</code> as the optional value of the <code>api</code> argument like so:</p>

<pre><code>http://yourdomain.com/fever/?api=xml</code></pre>

<p>The top level XML element is named <code>response</code>.</p>

<p>The response to each successfully authenticated request will have <code>auth</code> set to <code>1</code> and include at least one additional member:</p>

<ul>
<li><code>last_refreshed_on_time</code> contains the time of the most recently refreshed (not <em>updated</em>) feed  (Unix timestamp/integer)</li>
</ul>

<h2>Read</h2>

<p>When reading from the Fever API you add arguments to the query string of the API endpoint URL. If you attempt to <code>POST</code> these arguments (and their optional values) Fever will not recognize the request.</p>

<h3>Groups</h3>

<pre><code>http://yourdomain.com/fever/?api&amp;groups</code></pre>

<p>A request with the <code>groups</code> argument will return two additional members:</p>

<ul>
<li><code>groups</code> contains an array of <code>group</code> objects</li>
<li><code>feeds_groups</code> contains an array of <code>feeds_group</code> objects</li>
</ul>

<p>A <code>group</code> object has the following members:</p>

<ul>
<li><code>id</code> (positive integer)</li>
<li><code>title</code> (utf-8 string)</li>
</ul>

<p>The <code>feeds_group</code> object is documented under &#8220;Feeds/Groups Relationships.&#8221;</p>

<p>The &#8220;Kindling&#8221; super group is not included in this response and is composed of all feeds with an <code>is_spark</code> equal to <code>0</code>. The &#8220;Sparks&#8221; super group is not included in this response and is composed of all feeds with an <code>is_spark</code> equal to <code>1</code>.</p>

<h3>Feeds</h3>

<pre><code>http://yourdomain.com/fever/?api&amp;feeds</code></pre>

<p>A request with the <code>feeds</code> argument will return two additional members:</p>

<ul>
<li><code>feeds</code> contains an array of <code>group</code> objects</li>
<li><code>feeds_groups</code> contains an array of <code>feeds_group</code> objects</li>
</ul>

<p>A <code>feed</code> object has the following members:</p>

<ul>
<li><code>id</code> (positive integer)</li>
<li><code>favicon_id</code> (positive integer)</li>
<li><code>title</code> (utf-8 string)</li>
<li><code>url</code> (utf-8 string)</li>
<li><code>site_url</code> (utf-8 string)</li>
<li><code>is_spark</code> (boolean integer)</li>
<li><code>last_updated_on_time</code> (Unix timestamp/integer)</li>
</ul>

<p>The <code>feeds_group</code> object is documented under &#8220;Feeds/Groups Relationships.&#8221;</p>

<p>The &#8220;All Items&#8221; super feed is not included in this response and is composed of all items from all feeds that belong to a given group. For the &#8220;Kindling&#8221; super group and all user created groups the items should be limited to feeds with an <code>is_spark</code> equal to <code>0</code>. For the &#8220;Sparks&#8221; super group the items should be limited to feeds with an <code>is_spark</code> equal to <code>1</code>.</p>

<h3>Feeds/Groups Relationships</h3>

<p>A request with either the <code>groups</code> or <code>feeds</code> arguments will return an additional member:</p>

<p>A <code>feeds_group</code> object has the following members:</p>

<ul>
<li><code>group_id</code> (positive integer)</li>
<li><code>feed_ids</code> (string/comma-separated list of positive integers)</li>
</ul>

<h3>Favicons</h3>

<pre><code>http://yourdomain.com/fever/?api&amp;favicons</code></pre>

<p>A request with the <code>favicons</code> argument will return one additional member:</p>

<ul>
<li><code>favicons</code> contains an array of <code>favicon</code> objects</li>
</ul>

<p>A <code>favicon</code> object has the following members:</p>

<ul>
<li><code>id</code> (positive integer)</li>
<li><code>data</code> (base64 encoded image data; prefixed by image type)</li>
</ul>

<p>An example <code>data</code> value:</p>

<pre><code>image/gif;base64,R0lGODlhAQABAIAAAObm5gAAACH5BAEAAAAALAAAAAABAAEAAAICRAEAOw==</code></pre>

<p>The <code>data</code> member of a <code>favicon</code> object can be used with the <code>data:</code> protocol to embed an image in CSS or HTML. A PHP/HTML example:</p>

<pre><code>echo '&lt;img src="data:'.$favicon['data'].'"&gt;';</code></pre>

<h3>Items</h3>

<pre><code>http://yourdomain.com/fever/?api&amp;items</code></pre>

<p>A request with the <code>items</code> argument will return two additional members:</p>

<ul>
<li><code>items</code> contains an array of <code>item</code> objects</li>
<li><code>total_items</code> contains the total number of items stored in the database (added in API version 2)</li>
</ul>

<p>An <code>item</code> object has the following members:</p>

<ul>
<li><code>id</code> (positive integer)</li>
<li><code>feed_id</code> (positive integer)</li>
<li><code>title</code> (utf-8 string)</li>
<li><code>author</code> (utf-8 string)</li>
<li><code>html</code> (utf-8 string)</li>
<li><code>url</code> (utf-8 string)</li>
<li><code>is_saved</code> (boolean integer)</li>
<li><code>is_read</code> (boolean integer)</li>
<li><code>created_on_time</code> (Unix timestamp/integer)</li>
</ul>

<p>Most servers won’t have enough memory allocated to PHP to dump all items at once. Three optional arguments control determine the items included in the response.</p>

<ul>
<li><p>Use the <code>since_id</code> argument with the highest id of locally cached items to request 50 additional items. Repeat until the <code>items</code> array in the response is empty.</p></li>
<li><p>Use the <code>max_id</code> argument with the lowest id of locally cached items (or <code>0</code> initially) to request 50 previous items. Repeat until the <code>items</code> array in the response is empty. (added in API version 2)</p></li>
<li><p>Use the <code>with_ids</code> argument with a comma-separated list of item ids to request (a maximum of 50) specific items. (added in API version 2)</p></li>
</ul>

<h3>Hot Links</h3>

<pre><code>http://yourdomain.com/fever/?api&amp;links</code></pre>

<p>A request with the <code>links</code> argument will return one additional member:</p>

<ul>
<li><code>links</code> contains an array of <code>link</code> objects</li>
</ul>

<p>A <code>link</code> object has the following members:</p>

<ul>
<li><code>id</code> (positive integer)</li>
<li><code>feed_id</code> (positive integer) only use when <code>is_item</code> equals <code>1</code></li>
<li><code>item_id</code> (positive integer) only use when <code>is_item</code> equals <code>1</code></li>
<li><code>temperature</code> (positive float)</li>
<li><code>is_item</code> (boolean integer)</li>
<li><code>is_local</code> (boolean integer) used to determine if the source feed and favicon should be displayed</li>
<li><code>is_saved</code> (boolean integer) only use when <code>is_item</code> equals <code>1</code></li>
<li><code>title</code> (utf-8 string)</li>
<li><code>url</code> (utf-8 string)</li>
<li><code>item_ids</code> (string/comma-separated list of positive integers)</li>
</ul>

<p>When requesting hot links you can control the range and offset by specifying a length of days for each as well as a page to fetch additional hot links. A request with just the <code>links</code> argument is equivalent to:</p>

<pre><code>http://yourdomain.com/fever/?api&amp;links&amp;offset=0&amp;range=7&amp;page=1</code></pre>

<p>Or the first page (<code>page=1</code>) of Hot links for the past week (<code>range=7</code>) starting now (<code>offset=0</code>).</p>

<h3>Link Caveats</h3>

<p>Fever calculates Hot link temperatures in real-time. The API assumes you have an up-to-date local cache of items, feeds and favicons with which to construct a meaningful Hot view. Because they are ephemeral Hot links should not be cached in the same relational manner as items, feeds, groups and favicons.</p>

<p>Because Fever saves items and not individual links you can only "save" a Hot link when <code>is_item</code> equals <code>1</code>.</p>

<h2>Sync</h2>

<p>The <code>unread_item_ids</code> and <code>saved_item_ids</code> arguments can be used to keep your local cache synced with the remote Fever installation.</p>

<pre><code>http://yourdomain.com/fever/?api&amp;unread_item_ids</code></pre>

<p>A request with the <code>unread_item_ids</code> argument will return one additional member:</p>

<ul>
<li><p><code>unread_item_ids</code> (string/comma-separated list of positive integers)</p></li>
</ul>

<pre><code>http://yourdomain.com/fever/?api&amp;saved_item_ids</code></pre>

<p>A request with the <code>saved_item_ids</code> argument will return one additional member:</p>

<ul>
<li><code>saved_item_ids</code> (string/comma-separated list of positive integers)</li>
</ul>

<p>One of these members will be returned as appropriate when marking an item as read, unread, saved, or unsaved and when marking a feed or group as read.</p>

<p>Because groups and feeds will be limited in number compared to items, they should be synced by comparing an array of locally cached feed or group ids to an array of feed or group ids returned by their respective API request.</p>

<h2>Write</h2>

<p>The public beta of the API does not provide a way to add, edit or delete feeds or groups but you can mark items, feeds and groups as read and save or unsave items. You can also unread recently read items. When writing to the Fever API you add arguments to the <code>POST</code> data you submit to the API endpoint URL.</p>

<p>Adding <code>unread_recently_read=1</code> to your <code>POST</code> data will mark recently read items as unread.</p>

<p>You can update an individual item by adding the following three arguments to your <code>POST</code> data:</p>

<ul>
<li><code>mark=item</code></li>
<li><code>as=?</code> where <code>?</code> is replaced with <code>read</code>, <code>saved</code> or <code>unsaved</code></li>
<li><code>id=?</code> where <code>?</code> is replaced with the <code>id</code> of the item to modify</li>
</ul>

<p>Marking a feed or group as read is similar but requires one additional argument to prevent marking new, unreceived items as read:</p>

<ul>
<li><code>mark=?</code> where <code>?</code> is replaced with <code>feed</code> or <code>group</code></li>
<li><code>as=read</code></li>
<li><code>id=?</code> where <code>?</code> is replaced with the <code>id</code> of the feed or group to modify</li>
<li><code>before=?</code> where <code>?</code> is replaced with the Unix timestamp of the the local client&#8217;s most recent <code>items</code> API request</li>
</ul>

<p>You can mark the &#8220;Kindling&#8221; super group (and the &#8220;Sparks&#8221; super group) as read by adding the following four arguments to your <code>POST</code> data:</p>

<ul>
<li><code>mark=group</code></li>
<li><code>as=read</code></li>
<li><code>id=0</code></li>
<li><code>before=?</code> where <code>?</code> is replaced with the Unix timestamp of the the local client&#8217;s last <code>items</code> API request</li>
</ul>

<p>Similarly you can mark just the &#8220;Sparks&#8221; super group as read by adding the following four arguments to your <code>POST</code> data:</p>

<ul>
<li><code>mark=group</code></li>
<li><code>as=read</code></li>
<li><code>id=-1</code></li>
<li><code>before=?</code> where <code>?</code> is replaced with the Unix timestamp of the the local client&#8217;s last <code>items</code> API request</li>
</ul>
