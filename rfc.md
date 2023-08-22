# Lemmy Request for Change: Tag System

### Protocol:
According to https://www.w3.org/TR/activitystreams-vocabulary/#dfn-tags a general tag object exists in the ActivityPub protocol. 
Entirely unmoderated tags are not an option for lemmy as the moderation workload would be too much. Additionally users being able to type out tags themselves introduces splintering in the tag contents due to typos. A better solution is a curated list of tags users can attach to their posts. The list of tags can be maintained by both admins and moderators allowing for each community to tailor tags to their specific needs.
Content for both sets of tags would be located in their respective root:

- Instance Tags: `https://example.org/t/tag`
- Community Tags: `https://example.org/c/instance/t/tag`

The tag URL can then be utilized as a unique identifier for the tag, populating the `id` field of the tag object. Using the object ID optional tag federation can also be achieved, allowing for communities across multiple instances to share content via tags (example: News tag shared across instances). This would also solve the issue of splintered communities across instances while not forcing it on the communities in question.

To fill all needed use cases 3 tag types will be needed:

- NSFW
- Content Warning
- Generic Tag

Theoretically NSFW could be implemented using a preset "Content Warning" tag but seperating out this tag allows instances to better filter it out for moderation purposes (for example if no admin/moderator is willing to moderate NSFW content).
Both "NSFW" and "Content Warning" tags should blur the post body by default. Additionally the `sensitive` post flag to `true` should either of these types be present in the post tags to ensure correct handling on other fediverse platforms.

Example Tag Objects:

**Generic Tag:**
```json
{
    "id": "https://example.org/t/tag",
    "name": "Generic Tag"
}
```

**NSFW Tag:**
```json
{
    "type": "nsfw",
    "id": "https://example.org/t/tag",
    "name": "NSFW"
}
```

**Spoiler Tag:**
```json
{
    "type": "cw",
    "id": "https://example.org/t/tag",
    "name": "Spoiler"
}
```

**Remote Instance Spoiler Tag:**
```json
{
    "type": "cw",
    "id": "https://example.org/t/tag@example_remote.org",
    "name": "Spoiler"
}
```

Below is an example of a News post in the news community. The post has a communtiy tag for the newspaper company that published it, as well as a content warning for Blood/Gore. In order to share this news post across instances it also is tagged with the news tag of a remote instance.

Example Post json:

``` json
{
  "content": ". . .",
  "sensitive": true,
  "tag": [
    {
      "id": "https://example.org/c/news/t/newspaper_company_1",
      "name": "Newspaper Company Tag"
    },
    {
      "type": "cw",
      "id": "https://example.org/t/blood",
      "name": "Blood/Gore"
    },
    {
      "id": "https://example.org/t/news@example_remote.org",
      "name": "News"
    }
  ]
}
```

### Backend:
I am not quite familiar with the Lemmy Backend so any corrections or additions for this section are appreciated.

A new table "tags" would be required. This new table would consist of the columns `url`, `name`, `type`, `deleted`, `ìd` and `community_id`. Additional columns for tag display style (for example background & text color) could be added later by extending this table. The `community_id` field should be nullable, in which case the tag should be assumed to belong to the instance. This column will be primarily used to help list all tags belonging to the instance or a communtiy for moderation purposes (Adding/Editing/Removing tags).

Instance Admins should have the option to delete/ban tags.
Instances with nsfw disables/blocked could/should auto reject posts with an nsfw type tag.

The following API routes will need to be added:

**/tags:**
Parameters:

- community_id (optional) 
- auth (optional)

Returns:
List of all tags, if a community id is specified only tags of that communtiy are returned.
Deleted tags should not be returned in the list unless an admin/moderator.

```json
{
    "tags": [
        {
            "url": "https://example.org/t/tag",
            "name": "Generic Tag",
            "id": 1,
            "community_id": 1
        },
        {...}
    ]
}
```

**/tags/create:**
Parameters:

- name
- type
- community_id (optional)
- auth

Tag URL will be generated using the name and communtiy id.

Returns:
Object of freshly created tag

**/tags/edit:**
Parameters:

- id
- name
- type
- auth

Replaces tag name and type with the provided info. Community Id will not be consumed as a parameter as tags should not be community transferable.
The new Tag URL will be generated using the name and communtiy id.

Returns:
Object of freshly created tag

**/tags/delete:**
Parameters:

- id
- deleted
- auth

Deletes the tag associated with the id. Updates `deleted` field in table. Allows for un-deleting a tag.

Returns:
Object of freshly created tag


### Frontend:

**General Changes:**

Tags can be listed next to the Post title in the feed and at the end of the post in the post view.
Adding tags to a post can be implemented via a separate tag box where they can be typed in with search support. 

![tags post view iamge](https://github.com/Neshura87/Lemmy-RFC/blob/main/tags_post_view.png?raw=true)

Tags in the feed should be displayed next to the post interaction elements.
![tags feed view iamge](https://github.com/Neshura87/Lemmy-RFC/blob/main/tags_feed_view.png?raw=true)



**Moderator Changes:**

Moderators need a UI Section in the community/instance settings for adding, edditing and deleting tags. This could also implement a way to "import" a tag from another instance.
Moderators should also be able to add tags to user posts.


### Outlook:
There have already been several discussions about future expansions, in particular filtering tags has been requested. Given this proposal tag filtering in the feed view should be possible without further changes to the backend.