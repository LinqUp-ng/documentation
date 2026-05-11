# Menu Management API

This document provides the complete API implementation and integration guide for menu-related operations.

## API Implementation

```typescript
@/Users/ovieokomite-iffie/Documents/mobile apps/linqUp/api/handleMenu.ts:1-479
```

## Functions

### `insertMenu`
Inserts a new menu item into the database.

```typescript
const insertMenu = async (
  menuData: MenuInsert
): Promise<Response<any>> => {
  try {
    const insertData: any = {
      name: menuData.name,
      description: menuData.description,
      price: menuData.price,
      discount_price: menuData.discount_price,
      type: menuData.type || 'main',
      image_uri: menuData.image_uri,
      image_blur_hash: menuData.image_blur_hash,
      is_out_of_stock: menuData.is_out_of_stock ?? false,
      store_id: menuData.store_id,
    };

    const { data, error } = await supabase
      .from('menus')
      .insert(insertData)
      .select()
      .single();

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Menu item inserted successfully',
      data: data,
    };
  } catch (error) {
    console.error('Error inserting menu:', error);
    throw error;
  }
};
```

### `updateMenu`
Updates an existing menu item.

```typescript
const updateMenu = async (
  menuId: string,
  menuData: MenuUpdate
): Promise<Response<any>> => {
  try {
    const updateData: any = {
      name: menuData.name,
      description: menuData.description,
      price: menuData.price,
      discount_price: menuData.discount_price,
      type: menuData.type,
      image_uri: menuData.image_uri,
      image_blur_hash: menuData.image_blur_hash,
      is_out_of_stock: menuData.is_out_of_stock,
    };

    const { data, error } = await supabase
      .from('menus')
      .update(updateData)
      .eq('id', menuId)
      .select()
      .single();

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Menu item updated successfully',
      data: data,
    };
  } catch (error) {
    console.error('Error updating menu:', error);
    throw error;
  }
};
```

### `deleteMenu`
Soft deletes a menu item using a database function.

```typescript
const deleteMenu = async (menuId: string): Promise<Response<void>> => {
  try {
    const { error } = await (supabase.rpc as any)('delete_menu', {
      p_menu_id: menuId,
    });

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Menu item deleted successfully',
      data: undefined,
    };
  } catch (error) {
    console.error('Error deleting menu:', error);
    throw error;
  }
};
```

### `addMenuType`
Adds a new menu category type.

```typescript
const addMenuType = async (
  menuTypeData: MenuTypeInsert
): Promise<Response<any>> => {
  try {
    const { data, error } = await supabase
      .from('menu_types')
      .insert(menuTypeData)
      .select()
      .single();

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Menu type inserted successfully',
      data: data,
    };
  } catch (error) {
    console.error('Error inserting menu type:', error);
    throw error;
  }
};
```

### `getMenuTypeByName`
Retrieves a menu type definition by its name.

```typescript
const getMenuTypeByName = async (
  name: string
): Promise<Response<any>> => {
  try {
    const { data, error } = await supabase
      .from('menu_types')
      .select()
      .eq('name', name)
      .single();

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Menu type retrieved successfully',
      data: data,
    };
  } catch (error) {
    console.error('Error retrieving menu type by name:', error);
    throw error;
  }
};
```

### `getMenuTypeById`
Retrieves a menu type definition by its ID.

```typescript
const getMenuTypeById = async (
  id: string
): Promise<Response<any>> => {
  try {
    const { data, error } = await supabase
      .from('menu_types')
      .select()
      .eq('id', id)
      .single();

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Menu type retrieved successfully',
      data: data,
    };
  } catch (error) {
    console.error('Error retrieving menu type by ID:', error);
    throw error;
  }
};
```

### `getAllMenuTypes`
Retrieves all available menu types.

```typescript
const getAllMenuTypes = async (): Promise<Response<any>> => {
  try {
    const { data, error } = await supabase
      .from('menu_types')
      .select()
      .order('created_at', { ascending: false });

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Menu types retrieved successfully',
      data: data,
    };
  } catch (error) {
    console.error('Error retrieving menu types:', error);
    throw error;
  }
};
```

### `addMenuGroup`
Adds a new group (category of options) to a menu.

```typescript
const addMenuGroup = async (
  menuGroupData: MenuGroupInsert
): Promise<Response<any>> => {
  try {
    const { data, error } = await supabase
      .from('menu_groups')
      .insert(menuGroupData)
      .select()
      .single();

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Menu group inserted successfully',
      data: data,
    };
  } catch (error) {
    console.error('Error inserting menu group:', error);
    throw error;
  }
};
```

### `updateMenuGroup`
Updates an existing menu group.

```typescript
const updateMenuGroup = async (
  menuGroupId: string,
  menuGroupData: MenuGroupUpdate
): Promise<Response<any>> => {
  try {
    const { data, error } = await supabase
      .from('menu_groups')
      .update(menuGroupData)
      .eq('id', menuGroupId)
      .select()
      .single();

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Menu group updated successfully',
      data: data,
    };
  } catch (error) {
    console.error('Error updating menu group:', error);
    throw error;
  }
};
```

### `deleteMenuGroup`
Deletes a menu group.

```typescript
const deleteMenuGroup = async (
  menuGroupId: string
): Promise<Response<void>> => {
  try {
    const { error } = await supabase
      .from('menu_groups')
      .delete()
      .eq('id', menuGroupId);

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Menu group deleted successfully',
      data: undefined,
    };
  } catch (error) {
    console.error('Error deleting menu group:', error);
    throw error;
  }
};
```

### `insertSubMenu`
Adds a specific sub-menu item (option) to a group.

```typescript
const insertSubMenu = async (
  subMenuData: SubMenuInsert
): Promise<Response<any>> => {
  try {
    const { data, error } = await supabase
      .from('sub_menus')
      .insert(subMenuData)
      .select()
      .single();

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Sub menu inserted successfully',
      data: data,
    };
  } catch (error) {
    console.error('Error inserting sub menu:', error);
    throw error;
  }
};
```

### `updateSubMenu`
Updates an existing sub-menu item.

```typescript
const updateSubMenu = async (
  subMenuId: string,
  subMenuData: SubMenuUpdate
): Promise<Response<any>> => {
  try {
    const { data, error } = await supabase
      .from('sub_menus')
      .update(subMenuData)
      .eq('id', subMenuId)
      .select()
      .single();

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Sub menu updated successfully',
      data: data,
    };
  } catch (error) {
    console.error('Error updating sub menu:', error);
    throw error;
  }
};
```

### `deleteSubMenu`
Deletes a sub-menu item.

```typescript
const deleteSubMenu = async (
  subMenuId: string
): Promise<Response<void>> => {
  try {
    const { error } = await supabase
      .from('sub_menus')
      .delete()
      .eq('id', subMenuId);

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Sub menu deleted successfully',
      data: undefined,
    };
  } catch (error) {
    console.error('Error deleting sub menu:', error);
    throw error;
  }
};
```

### `getMenuByStoreID`
Retrieves a paginated list of menus for a specific store.

```typescript
const getMenuByStoreID = async (
  storeId: string,
  cursorId?: string | null,
  cursorCreatedAt?: string | null,
  limit = 20
): Promise<Response<Tables<'menus'>[]>> => {
    try {
        const { data, error } = await supabase.rpc('get_menu_by_store_id', {
            p_store_id: storeId,
            p_limit: limit,
            ...(cursorId && { p_cursor_id: cursorId }),
            ...(cursorCreatedAt && { p_cursor_created_at: cursorCreatedAt }),
        });

        if (error) throw error;

        return {
            isSuccessful: true,
            message: 'Menus retrieved successfully',
            data: data,
        };
    } catch (error) {
        console.error('Error retrieving menus by store ID:', error);
        throw error;
    }
};
```

### `getMenuItem`
Retrieves a single menu item with fully nested menu groups and sub-menus.

```typescript
const getMenuItem = async (
  menuId: string
): Promise<Response<CompositeTypes<'menu_item_response'>>> => {
  try {
    const { data, error } = await (supabase.rpc as any)('get_menu_item', {
      p_menu_id: menuId,
    });

    if (error) throw error;

    return {
      isSuccessful: true,
      message: 'Menu item retrieved successfully',
      data: data,
    };
  } catch (error) {
    console.error('Error retrieving menu item:', error);
    throw error;
  }
};
```

## Integration Examples

### Step 1: Upload Image (Required)
Before adding a menu item, you **must** upload its image to generate a storage URI and a blurhash. This ensures the app can display a beautiful placeholder while the high-resolution image loads.

Use the `uploadWithBlurhash` function from `handleUpload.ts`:

```typescript
import uploadApi from '@/api/handleUpload';

// id: current user or store ID
// uri: local file path from image picker
const uploadResult = await uploadApi.uploadWithBlurhash({
  id: storeId,
  uri: selectedImageUri,
  mimeType: 'image/jpeg',
  bucketName: 'menus',
  fileName: 'signature-dish'
});

if (uploadResult.isSuccessful) {
  const { uri, blur_hash } = uploadResult;
  // Proceed to Step 2
}
```

### Step 2: Add Menu Item
Pass the `uri` and `blur_hash` from Step 1 into the `insertMenu` function.

```typescript
import menuApi from '@/api/handleMenu';

const menuResult = await menuApi.insertMenu({
  name: "Signature Burger",
  description: "Our world-famous beef burger",
  price: 15.99,
  image_uri: uploadResult.uri, // From Step 1
  image_blur_hash: uploadResult.blur_hash, // From Step 1
  store_id: storeId,
  type: 'main'
});
```

### Get Menus by Store
```typescript
import menuApi from '@/api/handleMenu';

const result = await menuApi.getMenuByStoreID(storeId, null, null, 20);

if (result.isSuccessful) {
  const menus = result.data;
  // Use menus...
}
```

### Get Single Menu Item (Nested)
Retrieves a single menu item with all its associated groups and sub-menus.

```typescript
import menuApi from '@/api/handleMenu';

const result = await menuApi.getMenuItem(menuId);

if (result.isSuccessful) {
  const menu = result.data;
  console.log(menu.name);
  console.log(menu.menu_groups); // Array of groups with sub_menus
}
```
