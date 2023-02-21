   Select: 
    
     const [guests, count] = await this.guestRepository.guestRepo.findAndCount({
      where: where,
      // select: {
      //   id: true, createdAt: true,
      //   camp_guest_crm: {
      //     id: true,
      //     createdAt: true,
      //     campaign: {
      //       id: true,
      //       createdAt: true,
      //     }
      //   }
      // },
      relations: ['camp_guest_crm', 'camp_guest_crm.campaign'],
      take: page_size,
      skip: (page_no - 1) * page_size,
      order: {
        createdAt: 'DESC',
      },
    });
 
 Raw query: 
 
    const queryRunner = await AppDataSource.createQueryRunner();
    var result = await queryRunner.manager.query(
      `SELECT * FROM guests LIMIT 2`
    );
    await console.log(result)
